---
layout: post
title: OpenVPN client with username and password auth
---

Following on from last week’s post. We now need to setup the client.

### Network Manager
The easiest openvpn client is network-manager. If you are using Ubuntu run:

```source
# aptitude install network-manager-openvpn
# restart network-manager
```

* Now click on the network-manager applet, select configure VPN, and setup a new open-vpn connection.
* Set the gateway to your server
* Set the type to Password
* Point your CA to a copy of your server’s ca.crt and everything should just work

### Everything Else
Linux, Windows and OSX all have ports of OpenVPN, and I have setup the client on each of them. Unless you want to pay for Viscosity on the mac, the chances are you will need a client configuration file.

Attached is a simple client configuration file that will work. Edit it to match your server’s settings where appropriate. You will need this and your ca.crt in the same directory. On Windows the file extenion is “.ovpn”. On linux my file is called /etc/openvpn/client.conf

```
##############################################
# Sample client-side OpenVPN 2.0 config file.
# for connecting to multi-client server. 
##############################################

# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

dev tun
proto udp

# The hostname/IP and port of the server.
remote my-server-2.domain 1194


# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Try to preserve some state across restarts.
persist-key
persist-tun

# Certificate Authority
ca ca.crt

# Username/Password authentication is used on the server
auth-user-pass

# Verify server certificate by checking
# that the certicate has the nsCertType
# field set to "server".  This is an
# important precaution to protect against
# a potential attack discussed here:
#  http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the nsCertType
# field set to "server".  The build-key-server
# script in the easy-rsa folder will do this.
ns-cert-type server

# Set log file verbosity.
verb 3
```

On linux to start the openvpn client simply type:
```shell
# openvpn -config /etc/openvpn/client.conf
```

To manage the connection on Windows I would suggest using OpenVPN GUI. And either tunnelblick, or Viscosity (non-free) on OSX.
