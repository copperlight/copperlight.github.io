# Testing Ansible Galaxy Roles with Docker

<div class="meta">
  <span class="date"><small>2014-10-14</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

I learned a neat trick for testing [Ansible Galaxy](https://galaxy.ansible.com/)
[roles](http://docs.ansible.com/playbooks_roles.html) at
[AnsibleFest 2014](https://speakerdeck.com/mpdehaan/1-dot-8).

To get started with Docker on your Mac, install [VirtualBox](https://www.virtualbox.org/) and then
install [boot2docker](https://github.com/boot2docker/boot2docker), using [Homebrew](http://brew.sh/).
The boot2docker package will install docker as a dependency and it runs a small (24MB) Linux
VirtualBox virtual machine that provides a platform for running Docker images:

```
brew install boot2docker
boot2docker init
boot2docker up
$(boot2docker shellinit)
```

The Ansible Team (thanks, [Toshio!](https://github.com/abadger)) has
[released](https://github.com/ansible/ansible-docker-base/blob/master/examples/memcached-from-galaxy/site.yml)
Docker images that are included in the [Docker Hub Registry](https://registry.hub.docker.com/repos/ansible/),
which can be used for testing:

```
docker search ansible |grep ^ansible
ansible/ubuntu14.04-ansible                    Ubuntu 14.04 LTS with ansible                   12
ansible/centos7-ansible                        Ansible on Centos7                              11
```

There are two tags associated with each of these images: latest and devel.  The latest contains a
layered Ansible stable and devel contains a layered Ansible HEAD.  Pull one of these images from the
Docker Hub:

```
docker pull ansible/ubuntu14.04-ansible
```

This technique will allow you to rinse and repeat installs with different roles, which will help you
figure out which ones deliver the most suitable functionality for your use cases.

Launch the docker image and leave a shell process running:

```
docker run -i -t ansible/ubuntu14.04-ansible:stable /bin/bash
```

Download a role from Ansible Galaxy:

```
ansible-galaxy install geerlingguy.memcached
```

Create a local site.yml playbook to run the role:

```
- hosts: localhost
  roles:
      - role: geerlingguy.memcached
```

Execute the playbook:

```
root@ccca9e7f365f:/# ansible-playbook site.yml -c local

PLAY [localhost] **************************************************************

GATHERING FACTS ***************************************************************
ok: [localhost]

TASK: [geerlingguy.memcached | Install Memcached.] ****************************
skipping: [localhost]

TASK: [geerlingguy.memcached | Update apt cache.] *****************************
ok: [localhost]

TASK: [geerlingguy.memcached | Install Memcached.] ****************************
changed: [localhost]

TASK: [geerlingguy.memcached | Copy Memcached configuration.] *****************
changed: [localhost]

TASK: [geerlingguy.memcached | Ensure Memcached is started and set to run on startup.] ***
changed: [localhost]

NOTIFIED: [geerlingguy.memcached | restart memcached] *************************
changed: [localhost]

PLAY RECAP ********************************************************************
localhost                  : ok=6    changed=4    unreachable=0    failed=0

root@ccca9e7f365f:/#
```
