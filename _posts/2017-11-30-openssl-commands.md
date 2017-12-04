---
layout: post
title: Cert testing from the command line
---

I am always forgetting openssl command-line options. Just some quick commands
for testing the validity of ssl certs from the command line

**Checking local cert validity**
```shell
openssl x509 -in server.crt -text -noout
```

**Check a key**
```shell
openssl rsa -in server.key -check
```

**Check a remote cert**
```shell
$ echo | penssl s_client -servername www.example.com \
 -connect www.example.com:443 2>/dev/null | \
 openssl x509 -text```
```
