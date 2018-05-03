---
layout: post
title: Fedora/Exim/Gmail
---
Ever since working with Debian, I have wondered why more Linux distros don’t ship with exim on as the default mail client.

“exim -bt email@address” is reason enough

It is simpler to setup and easier to administer. If you want your fedora machine to send local email via Gmail, [here](http://elliotli.blogspot.com.au/2007/03/use-exim-with-gmail-smtp-on-fedora.html) is a guide.

One point to add, if you want root mail delivered somewhere, edit /etc/aliases and set an alias for root. Don’t forget to run “newaliases” when your done.
