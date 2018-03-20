---
layout: post
title: Centos 6, Corosync and Pacemaker for a simple Active/Passive cluster
---

Just playing with Corosync so I thought I would include a simple howto. For
examples sake, we will setup an OpenVPN server with a fail-over.

I set this up on 2 KVM guests using the default network configuration. In the
examples I will use the ip addresses of my guests, change them to match your
setup.

---|---|---
host1: | 192.168.122.20
host2: | 192.168.122.21
Clustered IP: | 192.168.122.23

# Prerequisites
## Software Packages
> Install packages on both machines.

You will need the EPEL repository setup.

```bash
# yum install http://mirror.iprimus.com.au/epel/6/i386/epel-release-6-7.noarch.rpm
```

Install the required packages
```bash
# yum install corosync pacemaker openvpn
```

You should set corosync to run on startup. But do not set openvpn to run.
Pacemaker will start OpenVPN when required.
```bash
# chkconfig corosync on
```

## Firewall
For corosync, use the following filtering. Port 5405 is used to receive multicast traffic.

```bash
iptables -I INPUT -p udp -m state --state NEW -m multiport --dports 5404,5405 -j ACCEPT
```

If you are installing OpenVPN allow udp traffic on 1194.

# Configure Corsync

## On our first host.

### Setup the corosync authkey
```bash
# corosync-keygen
Press keys on your keyboard to generate entropy.
Press keys on your keyboard to generate entropy (bits = 160).
```

> *Note*. If you are remotely logged in, no amount of key bashing will work. You need to be connected to console for key-smashing to result in entropy. One novel suggestion for generating IO I found on a forum, was to download the latest kernel source, untar and then run a find over the resulting directory tree.

Copy the file “/etc/corosync/authkey” to the other host. Set the permission to “0400”.

### corosync.conf

We will use the example config included. Copy this file and edit it.

```
$ cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf
```

We need to set bindnetaddr to our local subnet:

```
bindnetaddr: 192.168.122.0
```

And we need corosync to start pacemaker. Add the following to the end of the file

```
service {
	# Load the Pacemaker Cluster Resource Manager
	name: pacemaker
	ver: 0
}
```
### Start Corosync
```bash
# service corosync start
```
Once corosync has started successfully on the first host:

```
# ssh 192.168.122.21 -- service corosync start
```
Run “crm_mon” you should see that corosync is running on 2 nodes.

### Setting up Active/Passive Cluster
Disable Stonith:

```
crm configure property stonith-enabled=false
```

Add a cluster IP
```
crm configure primitive ClusterIP ocf:heartbeat:IPaddr2 params ip=172.25.3.20 cidr_netmask=21 op monitor interval=30s
```
Disable Failback (there is no need to failback in this case)
```
crm configure rsc_defaults resource-stickiness=100
```
With 2 nodes we cannot attain a quorum
```
crm configure property no-quorum-policy=ignore
```
Print the current config
```
crm configure show
```
You should now be able to ping you new cluster IP address 192.168.122.23. This ip address should remain available if you turn off corosync on the first host, or even shut it down.

## Adding a service
### Configure OpenVPN
This is not a post about setting up openvpn. I wrote about that previously,
though the post is specific to Debian. I have included my config file though.

Once configured copy the entire directory structure “/etc/openvppn” to the second host.
```
$ cat /etc/openvpn/openvpn.conf

# This is the clustered IP
local 192.168.122.23

dev tun
proto udp
port 1194

ca /etc/openvpn/easy-rsa/2.0/keys/ca.crt
cert /etc/openvpn/easy-rsa/2.0/keys/centosvm.crt
key /etc/openvpn/easy-rsa/2.0/keys/centosvm.key
dh /etc/openvpn/easy-rsa/2.0/keys/dh1024.pem

user openvpn
group openvpn
## You can make this any private subnet you like
server 192.168.123.0 255.255.255.0

persist-key
persist-tun

client-to-client

# make this connection the default gateway for network traffic
push "redirect-gateway def1"
push "dhcp-option DNS 192.168.122.1"

log-append /var/log/openvpn

plugin /usr/lib64/openvpn/plugin/lib/openvpn-auth-pam.so system-auth
client-cert-not-required
username-as-common-name
```
## Add Pacemaker rules
```
# crm configure primitive p_openvpn ocf:heartbeat:anything params binfile="/usr/sbin/openvpn" cmdline_options="--writepid /var/run/openvpn.pid --config /etc/openvpn/openvpn.conf --cd /etc/openvpn --daemon" pidfile="/var/run/openvpn.pid" op start timeout="20" op stop timeout="30" op monitor interval="20"
# crm configure commit
```
If all of this has worked you should now have an Active/Passive OpenVPN cluster.


