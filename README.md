What is Bastion host:
=====================

Bastion host is useful when we need to connect to number of our internal
servers from internet through that bastion host.
Bastion host just works like a jumphost.
Bastion host itself normally has limited port open. Only required services
are run in an bastion host to limit attackers hack the bastion host as
well as internal systems.

We will use bastion host, so that through the bastion host we can connect
to our internal servers.
It has couple of advantages. first one is, it saves the floating ip address.
Then later is, internal server does not need floating ip address to connect to them.

More info can be found on bastion host on https://en.wikipedia.org/wiki/Bastion_host

Background
=========
Here in this document, we will use proxycommand option of openssh to
connect our internal servers through bastion server.

first of all, we need netcat installed in our bastion server.
On rpm based system we can install nc by
```
$ sudo yum install -y nc
```
On the other hand, on deb based systems (debian, ubuntu) we can do following.
```
$ sudo apt-get install -y nc
```
Once, nc is installed, we need a ssh config file, which openssh client
will use when connecting to remote servers through bastion host.
An example of a fictitious ssh.config file can be following.
Also one predefined ssh.conf file included with this git repository. 
Please edit this file accordingly, before using.

ssh.config
=========
```
Host internal_server1
HostName 192.168.1.33
User internal_user1
ProxyCommand ssh -q user@bashtion.example.com nc %h %p

Host internal_server2
HostName 192.168.1.48
User internal_user2
ProxyCommand ssh -q user@bashtion.example.com nc %h %p
```
Explanation of the each options.
=========
```
Host: 			is the keyword.
internal_server1: 	is a user defined name. This name we will use when we ssh to the internal server.
HostName: 		either ip or fully qualified domain name of the internal server.
internal_user1: 	the user of the internal server.
User:			In the ProxyCommand line, user is the user of bashtion.example.com, bashion host.
nc:			is the netcat command
%h:			is host of internal server
%p:			is port.
```

Now we can use following command to connect internal servers through bastion host
by following way.

Usages
=========
```
$ ssh -F ssh.config internal_server1
[internal_user1@internal_server1 ~]$
```
Similarly, to connect to the second server.
```
$ ssh -F ssh.config internal_server2
[internal_user2@internal_server2 ~]$
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
$ ssh -L 8080:localhost:80 -F ssh.config internal_server1
```
Then we need to browse http:localhost:8080 in our browser which will go through
bastion host to destination, internal_server1.

2. ssh dynamic port forwarding.
To use dynamic port forwarding we need to use socks proxy server.
We will make our bastion host to accept port forwarding. Any port forwarding
through bastion will be forwarded to destination host.

In the local machines, we run following.
```
$ ssh -D 1080 bastion.example.com
```
Now we have to configure our browser's proxy setting. In firefox,
We will tick Manual proxy configuration. We will only put "localhost"
in the SOCKS Host field and port 1080. We will not touch any other field.
We need also set Firefox to use the DNS through that proxy, so even our DNS ... lookups are secure:

Type in about:config in the Firefox address bar
Find the key called "network.proxy.socks_remote_dns" and set it to true

The SOCKS proxy will stop working when we close our SSH session. We will need to change
these settings back to normal in order for Firefox to work again.
To make other programs use our SSH proxy server, we will need to configure each program in a similar way.

Now we can browse http://internal_server1 in our browser.

This way we can access all of our internal host and browse all of our
internal webservers.

Creating a host into a bastion host using ansible.
=======
We need to first clone the repository.
```
[cloud-user@khabiryubikeytest bastion]$ git clone https://github.com/khabiruddin/bastion.git
[cloud-user@khabiryubikeytest bastion]$ ls
README.md  bastion  bastion.yml  hosts  ssh.config
[cloud-user@khabiryubikeytest bastion]$ 
```
In this folder we will find a role bastion. Inside the bastion role folder, we need to edit the
bastion/var/main.yml file. In this file we will place username and public key following way. The 
following is an example only. Please change this file with your own username and corresponding public key.
```
[cloud-user@khabiryubikeytest vars]$ pwd
/home/cloud-user/bastion/bastion/vars
[cloud-user@khabiryubikeytest vars]$ cat main.yml
---
user:
   - { name: khabir, pubkey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQAcjs9FV8yqVXL+zxdc9U4i6FROHGd3SLXepYFu
+KwZNObG2nGl9jm5NIUKrP8VY3NDODsq1zpT1gMp6dkqFIbOqVZnd2bfodK05aMoGABt/CWZG9n3HX8iIN4lA4CMnKMawB67fQ
ztNcuVRhPKYmgNwrjHyz7OazdOxs5QtoMXbhwnL9GftZqGwKb2PvyWusiaf1gMVScwzJG2/1Qe82Us4uF7RllvuP8E+7c9TGVY
0AIMmrlZatn4ony+lcJGXYJkIIUJFCpDNkhKpHmxSNyeuuOCb2ii5' }
# vars file for bastion
[cloud-user@khabiryubikeytest vars]$
```
Ansible will read the username and public key from this file.
Now we will run the ansible-playbook. 
```
[cloud-user@khabiryubikeytest bastion]$ ansible-playbook bastion.yml
```
This will make this host a bastion host. Through this bastion host we can access internal machine.







