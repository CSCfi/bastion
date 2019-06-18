What is Bastion host:
=====================

Bastion host is useful when we need to connect to number of our internal
servers from internet through that bastion host.
Bastion host just works like a jumphost.
Bastion host itself normally has limited port open. Only required services
are run in an bastion host to limit attackers hack the bastion host as
well as internal systems.

We will create a bastion host, so that through the bastion host we can connect
to our internal servers. It has couple of advantages. 
First one is, it saves the floating ip address.
Then later is, internal server does not need floating ip address to connect to them.

More info can be found on bastion host on https://en.wikipedia.org/wiki/Bastion_host

Background
=========
Here in this document, we will use proxycommand option of openssh to
connect our internal servers through bastion server.

Also one predefined ssh.conf file included with this git repository. 
Please edit this file accordingly, before using.

An example of a fictitious ssh.config file can be following.

ssh.config
=========
```
Host bastion
  HostName 195.148.31.234
  User cloud-user

Host node1
  HostName 192.168.1.33
  User cloud-user
  ProxyCommand ssh -F ssh.config khuddin@195.148.31.234 nc %h %p

Host node2
  HostName 192.168.1.48
  User cloud-user
  ProxyCommand ssh -F ssh.config khabir@195.148.31.234 nc %h %p

Host *
  ControlPersist 30m
  GSSAPIAuthentication no
  StrictHostKeyChecking yes
```
Explanation of the each options.
=========
```
Host: 			is the keyword.
node1: 	      is a user defined name. This name we will use when we ssh to the internal server.
HostName: 		either ip or fully qualified domain name of the internal server.
cloud-user: 	the user of the node1.
User:			In the ProxyCommand line, user is the user of bashtion.example.com, bashion host.
nc:			is the netcat command
%h:			is host of internal server
%p:			is port.
```

Now we can use following command to connect internal servers through bastion host
by following way.

Usages
=========
Now time to access internal servers through bastion host. To do so we do following.
```
$ ssh -F ssh.config node1
[cloud-user@node1 ~]$
```
Similarly, to connect to the second server.
```
$ ssh -F ssh.config node2
[cloud-user@node2 ~]$
```
And this way, we can ssh other system in the internal network, just we need to edit
ssh.config file and add the internal systems information there.

Accessing web-pages through bastion host.
=========
We will use two methods to access webpages through the bastion host.

1. ssh local port forwarding.
We can use ssh local port forwarding technique to access web pages through bastion
host. To do that, we need to do following on local machine.
```
$ ssh -L 8080:localhost:80 -F ssh.config node1
```
Then we need to browse http:localhost:8080 in our browser which will go through
bastion host to destination, node1.

2. ssh dynamic port forwarding.
To use dynamic port forwarding we need to use socks proxy server.
We will make our bastion host to accept port forwarding. Any port forwarding
through bastion will be forwarded to destination host.

In the local machines, we run following.
```
$ ssh -D 1080 bastion
```
Now we have to configure our browser's proxy setting. In firefox,
We will tick Manual proxy configuration. We will only put "localhost"
in the SOCKS Host field and port 1080. We will not touch any other field.
We need also set Firefox to use the DNS through that proxy.

Type in about:config in the Firefox address bar
Find the key called "network.proxy.socks_remote_dns" and set it to true

Now we can browse http://node1 or http://node2 in our browser and so on.

This way we can access all of our internal host and browse all of our
internal webservers.

Creating a host into a bastion host using ansible.
=======
We can make a host into bastion host easily. We can use ansible to automate our all settings.
To make a host into bastion host, we need to first clone the repository.
```
[cloud-user@bastion]$ git clone https://github.com/khabiruddin/bastion.git
 
```
In this folder we will find a role bastion. Inside the bastion role folder, we need to edit the
bastion/var/main.yml file. In this file we will place username and public key following way. The 
following is an example only. Please change this file with your own username and corresponding public key.
```
[cloud-user@test vars]$ pwd
/home/cloud-user/bastion/bastion/vars
[cloud-user@test vars]$ cat main.yml
---
user:
   - { name: khabir, pubkey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQAcjs9FV8yqVXL+zxdc9U4i6FROHGd3SLXepYFu
+KwZNObG2nGl9jm5NIUKrP8VY3NDODsq1zpT1gMp6dkqFIbOqVZnd2bfodK05aMoGABt/CWZG9n3HX8iIN4lA4CMnKMawB67fQ
ztNcuVRhPKYmgNwrjHyz7OazdOxs5QtoMXbhwnL9GftZqGwKb2PvyWusiaf1gMVScwzJG2/1Qe82Us4uF7RllvuP8E+7c9TGVY
0AIMmrlZatn4ony+lcJGXYJkIIUJFCpDNkhKpHmxSNyeuuOCb2ii5' }
# vars file for bastion
[cloud-user@test vars]$
```
Similarly, we can put multiple username and corresponding public key in the same file.
Ansible will read the username and public key from this file.
Now we will run the ansible-playbook. 
```
[cloud-user@test bastion]$ ansible-playbook bastion.yml
```
This playbook will create a user khabir, and place khabir's public key in 
/home/khabir/.ssh/authorized_keys file so that user khabir can pass Through this bastion 
host to access internal machine.

Connecting to internal machines
==
Next part is we need to download ssh.config file from git repo in our local machine and then from the 
local machine we will connect to internal machines. If in the ssh.config file has following settings
```
Host node1
  HostName 192.168.1.33
  User cloud-user
  ProxyCommand ssh -F ssh.config khuddin@195.148.31.234 nc %h %p
```
then we can connect our internal_server1 though the bastion.example.com like below.
```
[local-user@local-machine /home]$ ssh -F ssh.config node1
[cloud-user@node1 /home/cloud-user] $
```
Architechture diagram
==
An architechture diagram has been included in this repo for the ease of understanding.

Running command and Installing packages through bastion host
==
We can run command and install packages in node1 or node2 through our bastion host by ansible.
To do so we need add parameters in ansible.conf file. An example of ansible.conf can be following.
```
╰$ cat ansible.cfg
[ssh_connection]
# Because I am using Yubikeys. Therefore I need to load the key along with ssh.
# Also I need to load ssh.config file as well.
ssh_args = -F ssh.config -I /usr/local/lib/opensc-pkcs11.so -o StrictHostKeyChecking=no
# If you dont use yubikey then you need use the following instead of above line.
# ssh_args = -F ssh.config -o StrictHostKeyChecking=no
[defaults]
roles_path    = roles/
library = /usr/share/ansible
ansible_python_interpreter = "/usr/bin/env python"
bin_ansible_callbacks = True
callback_plugins = callback_plugins/
callback_whitelist = timer
```
Also an inventory file can be like
```
╰$ cat hosts
[all]
bastion
node1
node2
```
Now we can run any playbook in node1 or node2 by following.
```
╰$ ansible-playbook -i hosts node1.yml
```
 







