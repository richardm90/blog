---
layout: post
title: "Testing GitHub Pages Locally"
date: 2024-03-04
modified_date: 2024-03-07
tags:
- GitHub Pages
- Jekyll
---

I started with the [Testing your GitHub Pages site locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)
instructions.

The following was all performed on my Linux Mint 21.3 machine.

## Install Jekyll

I used the standard [Jekyll installation instructions](https://jekyllrb.com/docs/installation/ubuntu/)
to install Jekyll on my Linux Mint machine.

This gave me the following:
* ruby 3.0.2p107
* gem 3.3.5
* gcc 11.4.0

## GitHub Pages Repo Setup

Create a `Gemfile` in the local repo directory.

```ruby
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins
gem "webrick", "~> 1.8"
```

Run `bundle install`.

## Serve the GitHub Pages Locally

```shell
# cd to the local repo directory
cd ~/git/rmss-docs
bundle exec jekyll serve
```

Now point your browser at [http://127.0.0.1:4000](http://127.0.0.1:4000).

```shell
# serve draft articles
bundle exec jekyll serve --drafts
```
