What is a bastion host
===
Bastion host just works like a jumphost, through which, we can connect to
other servers.

More info can be found on bastion host on https://en.wikipedia.org/wiki/Bastion_host

The benefit of using bastion hosts
===
It has some advantages. 
First of all, it saves floating IP addresses.
The internal server does not need a floating IP address to be connected to.
Moreover, a bastion host itself normally has a limited set of ports open. 
Only required services are run in a bastion host, this is to reduce the attack surface.
Because the bastion host has network connectivity to the internal systems it should be
secured properly.

How to create a bastion host
===
We can make one host as our bastion host.
To do so, we need to set up users in that host so that
through that host, users can go through and log in to other servers
and do administrative tasks.

After setting up users, we can use proxycommand option of ssh to 
connect to other servers behind the bastion host. We also need a predefined ssh.config file. 
An example ssh.config file has been included in this git repo.

How to configure a host into a bastion host
===
This git repo contains several ansible playbooks that make a host into a bastion host.
We need to change group_vars/all/users.yml. 
We also need to change the ssh.config file and populate the bastion host's
IP/FQDN and the other nodes' IP/FQDN.

Now we need to run following command from our localhost.

```
$ ansible-playbook -i inventories/hosts site.yml
```

Architecture diagram
==
An architecture diagram [Architecture.pdf](https://github.com/CSCfi/bastion/blob/master/Architecture.pdf) has been included in this repo for the sake of clarity. 
