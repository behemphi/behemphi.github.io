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
* Work with Chef's "data bag" construct

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

# Building a Rich Cookbook Ecosystem with Bershelf

We are going to revist the idea of user management on the nodes as a practical problem to solve.  To that end we will want to use some of what we have from our `sys_base` cookbook and refactor other things to be handled by community cookbooks for users and sudoers.

## Chef Supermarket

The [Chef Supemarket](https://supermarket.chef.io) is where you can go to find myriad community cookbooks.  

Because we prize simplicity, we want the chef node-name to match the hostname of the guest.  So we search on "hostname" in the supermarket.  After reading a bit of code, we chose [et_hostname](https://supermarket.chef.io/cookbooks/et_hostname) since it does exactly what we want it to do an all we need is to include it in our run_list.

## Working with Metadata

Rather than attempting to change the last article's cookbook as we would do in the real world, we are going to start with a fresh cookbook. The idea here is to help you keep the new concepts straight in your head. Refactoring should mean improving existing code rather than reinventing it.

In your `~/cookbooks` directory, create the cookbook `sys_base_berks`.  

```
behemphi:~ 20:42:49 > cd ~/cookbooks
behemphi:~/cookbooks 20:42:53 > chef generate cookbook sys_base_berks
behemphi:~/cookbooks 20:42:55 > vagrant init

```

Open the Vagrantfile and replace the contents with:

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: '10.42.42.11'

  config.vm.provision "chef_zero" do |chef|
    chef.channel = "stable"
    chef.cookbooks_path = "~/chef-base/cookbooks"
    chef.log_level = "warn"
    chef.node_name = "sys-base-berks"
    chef.nodes_path = "~/chef-base/nodes"
    chef.run_list = %w{ 
      recipe[sys_base_berks::default]
    }
    chef.version = "12.10.24"
    
  end

end
{% endhighlight %}

Open the the file `sys_base_berks/metadata.rb`  and add `depends` line.

{% highlight ruby %}
description 'Installs/Configures sys_base_berks'
license 'all_rights'
long_description 'Installs/Configures sys_base_berks'
maintainer 'The Authors'
maintainer_email 'you@example.com'
name 'sys_base_berks'
version '0.1.0'

# Set hostname of machine to be the same as the Chef node name.
depends 'et_hostname'
{% endhighlight %}

Notice that the file above is ordered differently than the default.  We are religious about alphabetizing configuration files.  It's a universal standard, so we apply it.

The exception here are the depends statements.  Those will come at the bottom of the file and, by convention, require a quick explanation for why it is included.  The depends statements will be arranged in alphabetical order by cookbook name.

According to [the documentation](https://supermarket.chef.io/cookbooks/et_hostname),  we should put this cookbook in our run_list.  Scroll back up and note its presense in our `Vagrantfile`.  

### Experiment on `vagrant up`

If you were to attempt to `vagrant up` at this moment, you would get an error.  Try it ...

**What Happened?**

Read through the error text carefully.  Note:

```
==> default: Missing Cookbooks:
==> default: ------------------
==> default: No such cookbook: hostname
```

During the compile phase, the `chef-client` run cannot find the dependency and stops right there .  This is the problem solved by Berkshelf.

We will fix this later in the article.

**--**

### Though Experiment on a Chef Run in the Wild

In a production environment, cookbooks are stored as versioned artifacts on the Chef Server.  

If you were to upload our `sys_base_berks` cookbook to a Chef Server and attempt to converge a node, you would get the same error In this case, during compilation, the `chef-client` will check in with the Chef Server for the `et_hostname` cookbook.  Assuming it has yet to be uploaded, it will not be found.

Berkshelf also eases this problem by allowing you to upload a cookbook and all of its dependencies.

### Going Deeper with Attributes

In [viewing the source](https://github.com/evertrue/et_hostname-cookbook)  for the cookbook `et_hostname`  a bit more we note this statement: "It is also recommended that you set the node['target_domain'] and node['target_fqdn'] attributes to a domain other than priv.evertrue.com." So we we take a look at the attributes file.

{% highlight ruby %}
default['target_domain'] = "priv.evertrue.com"
default['target_fqdn'] = "#{node.name}.#{node['target_domain']}"
override['domain'] = node['target_domain']
{% endhighlight %}

The first couple of lines define the aforementioned attributes we should set. For our purpose we are going to create an attribute file called `et_hostname.rb` to help a reader easily identify where these values are set.

```
behemphi:~/cookbooks/sys_base_berks 20:57:32 > chef generate attribute et_hostname
```
Our attribute file will look as follows:

{% highlight ruby %}
# attributes/et_hostname.rb

normal['target_domain'] = "kasasops.net"
normal['target_fqdn'] = "#{node.name}.#{node['target_domain']}"{% endhighlight %}

Notice the `normal` keyword.  Attributes identified this way have a higher precedence than `default` attributes.  During the compile phase of a `chef-client` run, the value of the `target_domain` attribute go from priv.evertrue.com to kasasops.net.

The `override` keyword is of the highest precedence.  

Chef provides a great deal of power here, but it comes at some complexity.  Take some time to familiarize yourself with [this list](https://docs.chef.io/attributes.html#attribute-precedence) from the Chef docs.  It will help you troubleshoot unexpected behaviors in the future.

**Gotcha!**

It may seem redundant to have the `normal['target_fqdn']` also set, however this is a must to get the desired effect.  Reasons are outside the scope of this article, but note the need to match precendence levels as a problem solving technique.

**--**

### The Default Recipe

Remember our standard for using the default recipe to call all other work.  We also apply this to the use of community cookbooks. Replace the contents of `recipes/default.rb` with:

{% highlight ruby %}
# recipes/default.rb

# set hostname and FQDN based on node name
include_recipe 'et_hostname::default'
{% endhighlight %}

We are still not ready to provision the node yet.

## Berkshelf 

To bring the dependency from the supermarket to our workstation, open `sys_base_berks/Berksfile` and replace its contents with: 

{% highlight ruby %}
source 'https://supermarket.chef.io'

metadata

cookbook 'et_hostname'
{% endhighlight %}

Note that all we have done is added the last line. So the file essentially says, "Get the et_hostname cookbook from the Chef Supermarket."

Now we will have Berkshelf install the dependency on our workstation: 

```
behemphi:~/cookbooks/sys_base_berks 21:17:20 > berks vendor ~/chef-base/cookbooks
behemphi:~/cookbooks/sys_base_berks 21:31:53 > ls -l ~/chef-base/cookbooks
```

You should see the et_hostname cookbook. 

**Gotcha!**

Now that you are using Berkshelf, you will need to remember to peform the `berks vendor ~/chef-base/cookbooks` command each time you make changes to cookbook code. This takes a (deep) copy of the cookbook and vendors it.

**--**

Now you can provision your node:

```
behemphi:~/cookbooks/sys_base_berks 22:14:47 > vagrant provision
[SNIP]
behemphi:~/cookbooks/sys_base_berks 22:15:02 > vagrant ssh -c 'cat /etc/hosts'
127.0.0.1   localhost.localdomain localhost
10.0.2.15 sys-base-berks.kasasops.net sys-base-berks
```

Check the chef output first to see where the desired state was written in the template.  Then run the vagrant command to see the file with your own eyes.  

Congratulations, you have successufully used a community cookbook. 

### Summary of Community Cookbook Usage Steps

There was a great deal of information there, so let's take a moment to summarize using a community cookbook:

1. Identify the cookbook you would like to use.
2. Minimally read README.md in the source. Ideally read recipes, templates and attributes to get an understanding of how it works.
3. In `metadata.rb` place a `depends` with the cookbook name
4. In `Berksfile` place a `cookbook` and the cookbook name
5. In `recipes/default.rb` call the cookbook in the same way the docs say to put it in the run_list.
6. Use the `berks` command to vendor your cookbook and the dependencies
7. Provision the node and sanity check.

## Composing More Dependencies

Our base cookbook is intended to install packages, NTP, users and their privileges, etc.  Using the steps above we are going to add those things now:


### NTP


### Users

#### Data Bags and Search

### Packages

Just like last article, we will also have some recipes of our own in this cookbook to do some things 
