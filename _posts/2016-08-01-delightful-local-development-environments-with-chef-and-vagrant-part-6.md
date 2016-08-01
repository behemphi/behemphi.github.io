---
categories: 
- Vagrant
- Chef
- Development Environment
date: 2016-08-01
layout: post
tags: 
- Vagrant
- Chef
- Development Environment
- DevOps
- OSX
- Berkshelf
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 6"
---

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management with Chef . 

# Table of Contents

1. [Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html) 
1. [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html)
1. [ChefDK and the Chef Provisioner](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html)
1. [Launching EC2 Instances with Knife](/vagrant/chef/development%20environment/2016/07/28/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)
1. [Cookbook Authoring](/vagrant/chef/development%20environment/2016/07/30/delightful-local-development-environments-with-chef-and-vagrant-part-5.html)
1. **[Using Community Cookbooks and Berkshelf](/vagrant/chef/development%20environment/2016/08/01/delightful-local-development-environments-with-chef-and-vagrant-part-6.html)**
1. [Testing Infrastructure with Test Kitchen]()
1. [Team Coding with Chef]()
1. [Multiple Guests with Vagrant]()

# Goals for this Article

At the end of this article you should be able to:

* Learn implement community cookbooks
* Use Berkshelf for dependency management

It may seem like a short list, but there is a great deal of ground to cover.  

# First Principles

For this article, a mental model of building an application cookbook important.

Consider a simple LEMP application on a single machine.  You might need to install and configure:

* Nginx
* MySQL
* PHP-FPM
* Monitoring Agent
* Logging Agent

Creating a single cookbook would mean you have solved the problem. However most of us run more than one system that depends on MySQL. We also deploy our monitoring agent to every machine that contains one of our applications.  

Chef cookbooks are designed to be composable. If you have experience with object oriented programming, this is the same composition from the famous maxim, "Favor composition over inheritance."  

If you are unfamiliar with that abstraction, consider a cookbook for each component of your system above.  In this case each component becomes reusable.  For example if you have a Tomcat application that also uses MySQL, would compose an application cookbook from cookbooks for:

* MySQL
* Monitoring Agent
* Logging Agent
* Tomcat

This can save many hours of work and significantly simplify operations.  

Equally important is Chef's notion of attribute precedence.  For any attribute defined in a cookbook A, if cookbook B includes cookbook A, then it can override the attribute.  This means that wil the right generalization of a monitoring agent installing it on every machine is frictionless.

This article makes these principles practical.


