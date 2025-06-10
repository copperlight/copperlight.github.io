---
authors:
  - copperlight
categories:
  - Ansible
date: 2014-06-30
draft: false
pin: false
---

# Ansible Vault and SSH Key Distribution

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

There are two types of SSH key distribution discussed in this post: private keys on local hosts and
public keys on remote hosts.  SSH private key distribution is best used for setting up your own
workstation or possibly an [Ansible Tower](http://www.ansible.com/tower) server.  In general, you
should not be distributing private keys widely; with a good SSH tunneling configuration and SSH
public key distribution, there should be no need for the private keys to be installed in more than
few places.  This configuration will show off a technique for configuring an SSH jump host bastion
that allows you to keep your private key on your own workstation; there is no need to have the SSH
private key on the bastion host.

For the purpose of this post, I have generated a new SSH key pair to demonstrate this technique;
this keypair is used nowhere.  Part of the trick to making this work is that the private key needs
to be base64 encoded so that line breaks are preserved when it is stored as a yaml string in the
`vars_files`.  Template files are created for the public and private keys to preserve file change
detection.

Generate a new key pair:

```bash
$ ssh-keygen -b 2048 -f junk_key -C junk
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in junk_key.
Your public key has been saved in junk_key.pub.
The key fingerprint is:
94:94:ae:ac:c3:5e:ee:7d:fa:2c:cb:0f:ae:19:c8:99 junk
The key's randomart image is:
+--[ RSA 2048]----+
|        ..       |
|       ...       |
|       .o        |
|       ..        |
|     . .S        |
|   . +o          |
|   .E.o .        |
|    +o *.o.      |
|   ..o=.*B+      |
+-----------------+
</pre>
```

Base64 encode the private key:

```bash
base64 -i junk_key > junk_key.b64
```

Create an Ansible vars_files yaml data file named `ssh_keys/ssh_key_vault.yml`.  The
`ssh_private_key` variable should contain the base64 encoded private key and the `ssh_public_key`
variable should contain the public key. Encrypt the file with `ansible-vault`:

```yaml
ssh_private_key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBeUtIRlhKWFJweXlCV2FobGExM2I5S2t1aGlwSHNkVkR4dDhaZnJrMWpqd1NnNEhCCmEzMzBnQnBxUFk1SkVxeEtPV0F3WWZleExwZ3VFcHk0Z2o5S1JxZGxTb0lsYllWbEtaUnY4RmhRSC9iT3lIKzAKVytJb0VzZ096MjR6U1ZQRU9ybWV6d3QzMzN0OWh0NDFsWVBBTHpzbkVaem9vVWE4ZTVKc1RzT0YzQzdmaUh3NApBSXZTOStWVUp5Mm8wUnZQN2ZMQkttV0FBN2dvWVA3d1Z6aVNQbEVrVVJIRGEyNXBVTmRTU1lxQzI2Y0c0UWNPCk14Q3VOeXdFRks0TGl5Q21zcHNXSnkzV3BkQ1FYQ0k1Q0J0SUVVTnR6Y1FpTFQvd0ZwbzRpRnp4NEREbkRsZWcKdkc2L1JHelFqMVJyeWRFdCtTNVdHenMzYkJHbDgxOWMvSmpPNVFJREFRQUJBb0lCQUYyQXZ5a3lEWDVheUlIUApjRXpFZG5Fa3M3RUZYVnBzcU9Tekx2K1hNM1Z4VzdOOE1uZDFRUkMrdnNxbldEamlvTWp5b2puV0pQWXhLQysyCmFHc1RNZnVSb2l4Q1VVMGtnUXdLeU14N2JBUXBreDl3SE05QnJDbHNvVEpkQ252ZkZUSEZObFVKNURqOEpYbEkKY0RLWkwyVVRyVmFSQ1AyNHFManllWldQbkFBTDZPc1JwSUc1Ukp1ays2QTVmVEppL0FVTmp4a2FIM1VOUklmTwpMZjYwOVJIUHZKUEtPNkNnNWVzK3RSY0VlbnR6ZVJxeENkYkh1b2NESjluUWNRQjVIVVBaeVdYOGwzODhJQ1hhCm5oaCt6VUhlYWY4cEM1dE9STGh0aDdsR1FFN3NOQ1FQRkovMHVCVWhleEVWREwxcU1Vd2JRZUpFU2orUmMrMi8KazRZeFhEVUNnWUVBNjhaNzRGUlJuUmViZy8xT2Z2K3ZlY0p2akEyRi9oWTBDVkpBOHdTcHdpNTlHTUpUS3g1TQpFWCtwNHBhMlY3NDZTZFFjY2l4K3hlWlhyZXkvbXhLWlByZnE0bVl1ZkJRRmVPT2tuWWdXTEhXUjV2cW1zUkFwCmZMQlhTbGdZNzV6SGFzSWJmOVlQZDhZQytLV1J4RlZyQml4eEtPQ2o1WFlrZmhoZkFnaDJsNnNDZ1lFQTJkZU4KM3FvR1lDM09GdllmWTJKVTQ4a2RvejNuT09uN09rd1pHNTFZOW5GM3JTYWpUeW9XRHpLTzc5MUNtK0hGNmhFWgpBWFlzeDlTcXJER1JYT2lvUi9ZMEpNVDZsS0ZuTUJpWERmRWROUVFCYStQQ3RFNWhqdTFoS1dPOHlIN21pZk1DCnQ0OHZBbGk5NEQxZjNxa2FjREtmRWVpa2VaazZaWWFhUWFTZ1k2OENnWUVBb0cxb3dzWjgxZWhIVURNZW96bDAKKytONkpSRGFtSDRoSUNxUXVRcjJPNE9JYVQxb2U5RmNyeGR2MEJiK3NZdGxlL0RRL2pzYWM2djlBd0l4aWVISQoxaTBzcktvY2ZSN2VibGh2SFNXSStPMXl2bmpVeld3UzNwM2FkMktrYlAzL2pydlBIRmZhSklSZVp6TzVrSjhTCmVKdnF6NGF5M3FKWnlGYnE1cVk5azRzQ2dZQmlzNVhtTTJkY0lLVG1KbkltVjZGYTYvN3Z2ZGFNSlFmZGJDbGMKSjdqdFFKQVc5aEM4aDdjaS82ZGY2d0tKR296UDl4czdYRTRCNU12SDVWV1ZvUnpPTGpHR0QzSHg4Z2VNOVRkTAo2OWx0OGZpcTU3R0tmSkViYjFhOHFDSWJQZFE2NE01MFdQM1Z0RnVqeEdzeHViRHU4U0M5dm9qM1I0UDhDRGJRClUwVVFwUUtCZ1FDc0pyMlBrdFl0QWJteHdCbUMrT1FVOFd4dHQwZ2ZoNGluZHpVV2J3MFZWY3Ivb3A5aDliekoKWG9TSVVSenQwUjZmNVVDK1RwQUdxY3ZPKzhtTUgwbnZpZ2VuYkhXdk01WFBuSUtwR2RLZmc4QkxUMUZnR2t3Ugpma2tydnpwWjM1dWpCTkRWdnl4ZWVCTyt2MTVqc0N1YTFTS0FsaXpzaWJrdE9lM1F1VTkwS3c9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

ssh_public_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIocVcldGnLIFZqGVrXdv0qS6GKkex1UPG3xl+uTWOPBKDgcFrffSAGmo9jkkSrEo5YDBh97EumC4SnLiCP0pGp2VKgiVthWUplG/wWFAf9s7If7Rb4igSyA7PbjNJU8Q6uZ7PC3ffe32G3jWVg8AvOycRnOihRrx7kmxOw4XcLt+IfDgAi9L35VQnLajRG8/t8sEqZYADuChg/vBXOJI+USRREcNrbmlQ11JJioLbpwbhBw4zEK43LAQUrguLIKaymxYnLdal0JBcIjkIG0gRQ23NxCItP/AWmjiIXPHgMOcOV6C8br9EbNCPVGvJ0S35LlYbOzdsEaXzX1z8mM7l junk
```

```bash
ansible-vault encrypt ssh_keys/ssh_key_vault.yml
Vault password:
Confirm Vault password:
Encryption successful
```

Create an inventory file named `inventory`, showing off the SSH jump host connection capability:

```ini
[localhost]
localhost ansible_connection=local

[group-all:children]
group-01
group-02

[group-01]
i-00000001 ansible_ssh_host=bastion+192.168.1.1 ansible_ssh_user=remoteuser
i-00000002 ansible_ssh_host=bastion+192.168.1.2 ansible_ssh_user=remoteuser

[group-02]
i-00000003 ansible_ssh_host=bastion+192.168.1.3 ansible_ssh_user=remoteuser
i-00000004 ansible_ssh_host=bastion+192.168.1.4 ansible_ssh_user=remoteuser
```

Create a template file for the private key named `templates/HOME_.ssh_junk`:

```jinja
{{ssh_private_key_decoded.stdout}}
```

Create a template file for the public key named `templates/HOME_.ssh_junk.pub`:

```jinja
{{ssh_public_key}}
```

Create a template file for the SSH jump host configuration named `templates/HOME_.ssh_config`:

```jinja
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 5

Host bastion
    User {{remote_user}}
    IdentityFile ~/.ssh/junk
    Hostname bastion

Host bastion+*
    User {{remote_user}}
    IdentityFile ~/.ssh/junk
    ProxyCommand ssh -T -a bastion nc $(echo %h |cut -d+ -f2) %p 2>/dev/null
    StrictHostKeyChecking no
```

This configuration assumes that you have a consistent remote username defined on the bastion server
and your protected hosts.

Write a playbook to install the SSH key and configuration on your local workstation named
`config_local-ssh.yml`:

```yaml
- name: configure local ssh
  hosts:
  - localhost
  gather_facts: false
  sudo: false
  vars:
    local_home: "{{ lookup('env','HOME') }}"
    local_user: "{{ lookup('env','USER') }}"
    remote_user: remoteuser
  vars_files:
  - ssh_keys/ssh_key_vault.yml
  tasks:
  - file: path={{local_home}}/.ssh state=directory mode=0700 owner={{local_user}}

  - template: src=templates/HOME_.ssh_config dest={{local_home}}/.ssh/config mode=0644 owner={{local_user}} backup=yes

  - shell: echo {{ssh_private_key}} |base64 --decode
    register: ssh_private_key_decoded

  - template: src=templates/HOME_.ssh_junk dest={{local_home}}/.ssh/junk mode=0600 owner={{local_user}}

  - template: src=templates/HOME_.ssh_junk.pub dest={{local_home}}/.ssh/junk.pub mode=0644 owner={{local_user}}
```

Run the playbook to setup your local workstation with SSH keys and configuration:

```bash
ansible-playbook -i inventory config_local-ssh.yml --ask-vault-pass
Vault password:
```

Test your SSH tunneling access to a remote host behind the bastion server:

```bash
ssh bastion+192.168.1.1
ssh bastion+192.168.1.1 "date; date > /tmp/date.out"
scp bastion+192.168.1.1:/tmp/date.out .
```

Notice that the hosts behind the bastion server are referenced in the SSH command the same way that
they are referenced in the Ansible inventory file.  The "+" character used as a separator was
selected explicitly for its ability to be used interchangeably at the command line and in the
Ansible inventory.  IP addresses are being used on the right hand side of the expression since the
secondary connection to the protected host relies on the name resolution capabilities of the first
host in the tunnel.  If you had a reliable dynamic DNS service that was keeping up with changes to
the protected hosts and was accessible to the bastion host, then you could use host names instead,
such as `bastion+webserver01`.  This host selection technique can be extended to an Ansible dynamic
inventory script, if you were running instances at a cloud provider such as AWS.  When you write a
[dynamic inventory script](http://docs.ansible.com/developing_inventory.html), the data format
should look like this:

```json
{
    "group-01": {
        "hosts": [
            "i-00000001",
            "i-00000002"
        ]
    },
    "group-02": {
        "hosts": [
            "i-00000003",
            "i-00000004"
        ]
    },
    "group-all": {
        "children": [
            "group-01",
            "group-02"
        ]
    },
    "localhost": [
        "localhost"
    ],
    "_meta": {
        "hostvars": {
            "i-00000001": {
                "ansible_ssh_host": "bastion+192.168.1.1",
                "ansible_ssh_user": "remoteuser"
            },
            "i-00000002": {
                "ansible_ssh_host": "bastion+192.168.1.2",
                "ansible_ssh_user": "remoteuser"
            },
            "i-00000003": {
                "ansible_ssh_host": "bastion+192.168.1.3",
                "ansible_ssh_user": "remoteuser"
            },
            "i-00000004": {
                "ansible_ssh_host": "bastion+192.168.1.4",
                "ansible_ssh_user": "remoteuser"
            },
            "localhost": {
                "ansible_connection": "local"
            },
        }
    }
}
```

The nice thing about this style of SSH configuration is that you can have multiple bastion hosts in
different locations and target the hosts behind each of them, provided that you give your bastion
hosts different names.  The method of accessing them is the same between direct SSH connections and
Ansible execution.  Once you have this infrastructure in place, you can start distributing public
SSH keys to your protected hosts.

Write a `templates/etc_sudoers` file that grants NOPASSWD access to the sudo group:

```
Defaults    env_reset
Defaults    mail_badpass
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=NOPASSWD: ALL

#includedir /etc/sudoers.d
```

Write a playbook `update_remote-ssh.yml` to configure NOPASSWD sudo access for your remote user and
distribute SSH public keys on your remote hosts.  This will allow subsequent playbook execution to
operate more easily against your remote hosts.  In order for this to work, the paramiko connection
type must be used initially, so that the password can be requested once and re-used across all hosts.

```yaml
- name: update remote ssh
  hosts:
  - group-all
  gather_facts: false
  sudo: true
  connection: paramiko
  vars_files:
  - ssh_keys/ssh_key_vault.yml
  tasks:
  - copy: src=templates/etc_sudoers dest=/etc/sudoers mode=0440 owner=root group=root

  - user: name=remoteuser groups=sudo shell=/bin/bash state=present

  - authorized_key: user=remoteuser state=present key={{ssh_public_key}}
```

Run an SSH configuration playbook against remote hosts through the SSH tunnel, providing the SSH
password, sudo password and vault password:

```bash
ansible-playbook -i inventory update_remote-ssh.yml --ask-pass --ask-sudo-pass --ask-vault-pass
SSH password:
sudo password [defaults to SSH password]:
Vault password:

PLAY [update ssh] *************************************************************

TASK: [copy src=templates/etc_sudoers dest=/etc/sudoers mode=0440 owner=root group=root] ***
ok: [i-00000001]
ok: [i-00000002]
ok: [i-00000003]
ok: [i-00000004]

TASK: [user name=remoteuser groups=sudo shell=/bin/bash state=present] ***
ok: [i-00000001]
ok: [i-00000002]
ok: [i-00000003]
ok: [i-00000004]

TASK: [authorized_key user=remoteuser state=present key="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIocVcldGnLIFZqGVrXdv0qS6GKkex1UPG3xl+uTWOPBKDgcFrffSAGmo9jkkSrEo5YDBh97EumC4SnLiCP0pGp2VKgiVthWUplG/wWFAf9s7If7Rb4igSyA7PbjNJU8Q6uZ7PC3ffe32G3jWVg8AvOycRnOihRrx7kmxOw4XcLt+IfDgAi9L35VQnLajRG8/t8sEqZYADuChg/vBXOJI+USRREcNrbmlQ11JJioLbpwbhBw4zEK43LAQUrguLIKaymxYnLdal0JBcIjkIG0gRQ23NxCItP/AWmjiIXPHgMOcOV6C8br9EbNCPVGvJ0S35LlYbOzdsEaXzX1z8mM7l junk"] ***
ok: [i-00000001]
ok: [i-00000002]
ok: [i-00000003]
ok: [i-00000004]


PLAY RECAP ********************************************************************
i-00000001               : ok=3    changed=0    unreachable=0    failed=0
i-00000002               : ok=3    changed=0    unreachable=0    failed=0
i-00000003               : ok=3    changed=0    unreachable=0    failed=0
i-00000004               : ok=3    changed=0    unreachable=0    failed=0
```

The best way to keep Ansible output concise is to run without verbosity -- only crank this up if you
need it to diagnose a problem.
