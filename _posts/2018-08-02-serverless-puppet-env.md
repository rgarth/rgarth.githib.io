---
layout: post
title: Using vagrant to build a serverless puppet host for testing
---
I spent a little bit of time setting up a repo which will build a serverless
EL7 puppet test box.

[https://github.com/rgarth/puppetlab]()

It's fairly straight forward.
- Vagrant pulls latest Centos7
- bootstrap.sh setups the initial environment
- Puppet is run from cron, or manually `/usr/local/bin/puppet-apply`
- Puppet code lives in `./puppet`
- To sync changes from the host to the guest: `vagrant rsync`

I just find it a quick way to test modules out in a clean environment.
