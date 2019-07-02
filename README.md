What is Bastion host
====
Bastion host just works like a jumphost, through which, we can connect
our internal servers.

More info can be found on bastion host on https://en.wikipedia.org/wiki/Bastion_host

Benefit of using Bastion host
===
It has some of advantages. 
First of all, it saves the floating ip address.
Then, internal server does not need floating ip address to connect to them.
Moreover, Bastion host itself normally has limited port open. Only required services
are run in an bastion host to limit attackers hack the bastion host as
well as internal systems.

How to create a bastion host
===
We can make one host as our bastion host.
To do so, we need to set up users in that host so that
through that host, users can go through and log in to other servers
and do administrative tasks.

After setting up users in host, we can use proxycommand option of ssh to 
connect other servers through bastion host. We also need a predefined ssh.conf file  
An example ssh.config file has been included in this git repo.

How to configure a host into bastion host
===

Architechture diagram
==
An architechture diagram Architecture.pdf has been included in this repo for the ease of understanding. 

 







