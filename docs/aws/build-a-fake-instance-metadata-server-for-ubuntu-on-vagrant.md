# Build a Fake Instance Metadata Server for Ubuntu on Vagrant

<div class="meta">
  <span class="date"><small>2014-07-22</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

Let's say that you have a proper build/bake/deploy pipeline in place for running appications in AWS.
This is a reliable way to deploy applications at scale and move traffic between different versions
of applications as a part of the deployment pipeline.  However, the whole process may take 20-40
minutes or so to complete for any particular build.  If you want to iterate more rapidly on your
development efforts, you could skip the full process with a quickpatch ssh/rsync deployment to a
single server or you could stand up a local Vagrant base image and iterate on that.  Now let's say
that the application you are working on is intended to work with instance metadata, particularly for
the purpose of obtaining a rotating set of access and secret keys.  It might be nice to have a fake
metadata service running on your local Vagrant image so that you can test your application in a
manner similar to how it will be running in the cloud.  In this post, I describe how to build and
configure a fake metadata service for an Ubuntu image running on Vagrant.

# Package Layout

This layout assumes that you will be installing custom packages to the `/apps` directory, and there
is a [daemontools](http://cr.yp.to/daemontools.html) service hierarchy located at `/service`.  The
`fake-metadata/service/run` file is a script suitable for use with daemontools.

```
root/
├── apps
│   └── fake-metadata
│       ├── app.py
│       └── service
│           └── run
└── etc
    ├── init.d
    │   └── fake-metadata
    ├── logrotate.d
    │   └── fake-metadata
    └── network
        └── iptables.rules
```

# Building and Packaging

For cross-platform build packaging, it will be easiest to use the
[nebula ospackage](https://github.com/nebula-plugins/gradle-ospackage-plugin) plugin with Gradle.
With this plugin available, your build script will look something like this:

```groovy
apply plugin: 'nebula-ospackage'

ospackage {
    version='1.0'
    packageName='fake-metadata'

    requires('python-flask')

    link('/apps/fake-metadata/logs', '/mnt/logs/fake-metadata')
    link('/service/fake-metadata', '/apps/fake-metadata/service')
}

buildDeb {
    postInstall file('scripts/postInstall.sh')
    preUninstall 'svc -d /service/fake-metadata'
    postUninstall file('scripts/postUninstall.sh')
}

task build(dependsOn: ['buildDeb'])
```

# Fake Metadata Application

The simplest approach to building the service is to create a [Flask](http://flask.pocoo.org/)
service and have it run bare on the Vagrant instance.  Given how small it will be and limited amount
of traffic it will need to serve, there is no need to run this behind a dedicated static webserver
like nginx or Apache.  The nice thing about using Flask and having a basic structure in place is
that it is then easy to extend the application to add other endpoints when needed.

```
#!/usr/bin/env python

from flask import Flask, jsonify, abort, make_response, request
import os
import sys


app = Flask(__name__)


BaseIAMRole = {
  'Code': 'Success',
  'LastUpdated' : '',
  'Type': 'AWS-HMAC',
  'AccessKeyId': '',
  'SecretAccessKey': '',
  'Token': '',
  'Expiration': ''
}


@app.route('/', methods = ['GET'])
def index():
    return 'latest'


@app.route('/latest/', methods = ['GET'])
def latest():
    return 'meta-data'


@app.route('/latest/meta-data/', methods = ['GET'])
def meta_data():
    endpoints = [
        'iam',
        'public-hostname',
        'public-ipv4'
    ]
    return ('\n').join(endpoints)


@app.route('/latest/meta-data/iam/', methods = ['GET'])
def iam():
    return 'security-credentials'


@app.route('/latest/meta-data/iam/security-credentials/', methods = ['GET'])
def security_credentials():
    return 'BaseIAMRole'


@app.route('/latest/meta-data/iam/security-credentials/BaseIAMRole', methods = ['GET'])
def base_iam_role():
    # update the payload to contain a current set of accesss and secrey keys
    return jsonify(BaseIAMRole)


@app.route('/latest/meta-data/public-hostname', methods = ['GET'])
def public_hostname():
    return os.environ['EC2_LOCAL_HOSTNAME']


@app.route('/latest/meta-data/public-ipv4', methods = ['GET'])
def public_ipv4():
    return os.environ['EC2_LOCAL_IPV4']


@app.errorhandler(400)
def not_found(error):
    return make_response(jsonify( { 'error': 'bad request' } ), 400)


@app.errorhandler(404)
def not_found(error):
    return make_response(jsonify( { 'error': 'not found' } ), 404)


@app.errorhandler(500)
def not_found(error):
    return make_response(jsonify( { 'error': 'internal server error' } ), 500)


if __name__ == '__main__':
    if len(sys.argv) > 1:
        if ':' in sys.argv[1]:
            host=sys.argv[1].split(':')[0]
            port=int(sys.argv[1].split(':')[1])
            app.run(host=host, port=port)
        else:
            app.run(host=sys.argv[1])
    else:
        app.run(debug=True)
```

# Daemontools Run Script

This script is watched by the supervise process, which then starts (or restarts) the application if
it is not running.  Switching to a non-root user and redirecting output to the log file occurs here.

```
#!/bin/bash

ulimit -n 32768

source /etc/profile.d/environment.sh

export PATH=/command:/usr/local/bin:/usr/local/sbin:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/scripts

if [ ! -d "/mnt/logs/fake-metadata" ]; then
  mkdir -p /mnt/logs/fake-metadata
  chmod 0777 /mnt/logs/fake-metadata
fi

USER=someuser
PYTHON=/usr/bin/python
APP=/apps/fake-metadata/app.py
OPTS=127.0.0.1:8000
LOG=/mnt/logs/fake-metadata/server.log

echo "starting fake-metadata"
exec setuidgid $USER $PYTHON $APP $OPTS >> $LOG 2>&1
```

# PostInstall Script

This is where most of the trickiness occurs.

The post install script is responsible for modifying the `/etc/network/interfaces` file, adding the
metadata server IP address and configuring iptables in an idempotent manner.  When the post install
script is packaged by the nebula ospackage plugin into a Debian package, it gets a `#!/bin/sh -e`
shebang invocation, which means that the script will halt execution at any point where it evaluates
a non-zero return code.  This means that the script needs to be written such a way that the
environment state testing being done always returns a true value so that the script does not fail,
hence the `||true` constructs.

We are attaching an extra IP address to the loopback interface, so we need to redirect traffic from
169.254.169.254:80 to the location where the fake metadata server is running.  We are dealing with
the loopback interface, which means that the PREROUTING nat table is never hit and we must use the
OUTPUT table instead.  You cannot DNAT packets destined for the loopback interface, because the
kernel will treat them as martians and drop them, so you must REDIRECT the packets to the desired
port.  When performing the redirection from port 80 to 8000 on the loopback interface, it sends the
packets to 127.0.0.1:8000, not 169.254.169.254:8000, so the fake metadata server must be listening
on localhost port 8000.

If for some reason, you need to troubleshoot the post install script, it can be found at
`/var/lib/dpkg/info/fake-metadata.postinst` following an attempted package installation.


```bash
LINE=$( grep 169.254.169.254/32 /etc/network/interfaces || true )
if [[ ! "$LINE" == *169.254.169.254/32*  ]]; then
    sed -i '/iface lo inet loopback/a up ip addr add 169.254.169.254/32 dev lo scope host' /etc/network/interfaces
fi

LINE=$( /sbin/ip addr |grep 169.254.169.254/32 || true )
if [[ ! "$LINE" == *169.254.169.254/32*  ]]; then
    /sbin/ip addr add 169.254.169.254/32 dev lo scope host
fi

LINE=$( grep iptables-restore /etc/network/interfaces || true )
if [[ ! "$LINE" == *iptables-restore*  ]]; then
    sed -i '/up ip addr add/a pre-up iptables-restore < /etc/network/iptables.rules' /etc/network/interfaces
fi

LINE=$( iptables -t nat -L |grep 8000 || true )
if [[ ! "$LINE" == *8000*  ]]; then
    iptables -t nat -A OUTPUT -p tcp -d 169.254.169.254/32 --dport 80 -j REDIRECT --to-ports 8000
fi

/usr/sbin/update-rc.d fake-metadata defaults
```

# PostUninstall Script

This script is the inverse of the post install script; it returns the system to its previous state.

```bash
LINE=$( grep 169.254.169.254/32 /etc/network/interfaces || true )
if [[ "$LINE" == *169.254.169.254/32*  ]]; then
    sed -i '/up ip addr add 169.254.169.254\/32 dev lo scope host/d' /etc/network/interfaces
fi

LINE=$( /sbin/ip addr |grep 169.254.169.254/32 || true )
if [[ "$LINE" == *169.254.169.254/32*  ]]; then
    /sbin/ip addr delete 169.254.169.254/32 dev lo scope host
fi

LINE=$( grep iptables-restore /etc/network/interfaces || true )
if [[ "$LINE" == *iptables-restore*  ]]; then
    sed -i '/pre-up iptables-restore < \/etc\/network\/iptables.rules/d' /etc/network/interfaces
fi

LINE=$( iptables -t nat -L |grep 8000 || true )
if [[ "$LINE" == *8000*  ]]; then
    iptables -t nat -D OUTPUT -p tcp -d 169.254.169.254/32 --dport 80 -j REDIRECT --to-ports 8000
fi

/usr/sbin/update-rc.d -f fake-metadata remove
rm -rf /apps/fake-metadata
pkill -f 'supervise fake-metadata'
```

# Log Rotation

To keep the local Vagrant instance clean, it is useful to configure log rotation.  Sending a HUP
signal to the service allows it to continue writing to the new logfile.

```bash
/mnt/logs/fake-metadata/server.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 644 root root
    postrotate
        svc -h /service/fake-metadata
    endscript
}
```
