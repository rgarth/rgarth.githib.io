---
layout: post
title: Running Jekyll/gh-pages locally
---
Running jekyll locally can be great for testing Github pages before pushing to
your account, and it's pretty simple. The following worked on fedora. This
assumes you already have a gh-pages repo and you are in the toplevel directory
of that repository.

#### Install ruby bundler
```bash
$ gem install bundler
```

#### Create the Gemfile for thie project
```bash
$ cat <<EOT > gemfile
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
EOT
```

#### Install and run Jekyll
```bash
$ bundle install
$ jekyll serve
```

You can now head to http://localhost:4000 and see your site
