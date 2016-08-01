---
categories: 
- Geekery
date: 2013-06-07
layout: post
tags: 
- system administration 
- command line
- vi
- vim
- mysql
- postgres
- python 
title:  "Command Line with Vi Behaviors"
---

I am not a fan of vi. I do, however, recognize that it is the one editor I can count on to be on a *nix distribution so I am pretty good with it. I wanted my command line to allow me to edit in the same style. That meant adding the below line to my ~/.profile on Ubuntu Lucid:

set -o vi

I love this. Esc + ^ returns me to first character of the line. Esc + bbbbb jumps me back 5 words instead of having to go character by character. You get the idea.

What about the other CLI tools like the MySQL client? The Python interactive editor? Postgres client? Ruby’s irb editor? Well it turns out that if the tool uses readline, then you can get the same convenience. In the file ~/.inputrc put the line:

set editing-mode vi

You can now navigate your command line client in vi-centric ways.

The idea for all this came from [here](http://askubuntu.com/questions/9506/vi-in-mysql-for-command-editing).