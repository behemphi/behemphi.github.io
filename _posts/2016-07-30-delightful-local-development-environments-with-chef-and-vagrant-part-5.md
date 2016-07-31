---
categories: 
- Vagrant
- Chef
- Development Environment
date: 2016-07-28
layout: post
tags: 
- Vagrant
- Chef
- Development Environment
- DevOps
- OSX
- Berkshelf
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 5"
---

This is the second post in a series of article describing how to set up a local development environment for a simple microservice architecture using Vagrant and Chef.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management .  

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

# Table of Contents

1. [Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html) 
1. [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html)
1. [ChefDK and the Chef Provisioner](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html)
1. [Launching EC2 Instances with Knife](/vagrant/chef/development%20environment/2016/07/28/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)
1. **[Cookbook Authoring](/vagrant/chef/development%20environment/2016/07/30/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)**
1. [Using Community Cookbooks and Berkshelf]()
1. [Testing Infrastructure with Test Kitchen]()
1. [Team Coding with Chef]()
1. [Multiple Guests with Vagrant]()

# Goals for this Article

At the end of this article you should be able to:

* Understand the basic components of cookbooks
* Write a recipe
* Write a template
* Set attributes
* Create a base cookbook of configuration you would expect for every machine (node) in your fleet

# First Principles

For this article it is important to make the following distinctions:

* "Cookbook" is a unit of work for installing, configuring and running a service. (e.g. Nginx)
* "Recipes" are contained in a cookbook. They are the code execution path.
* "Templates" are dynamic files meant to be written to disk during the converge phase of chef-client run. 
* "Attributes" are an abstraction for setting sane configuration default configuration parameters while allowing for uses of a cookbook to intelligently override them.

This article is very heavy on concepts within a cookbook.  Relationships between cookbooks is covered in a future article.

We will not yet deal with concepts that require the Chef Server such as environments, roles and data bags.

# Conceptualizing a Cookbook

We are going to use a base configuration cookbook to ensure there are some tools, scripts and settings on every node we configure with Chef.  

To this end we will create recipes to: 

* update the package system and install the packages we want
* create a group from limited privilege users
* configure sshd based on an attribute and a template

## Scaffolding a Cookbook

On your workstation create a workspace for your custom authored Chef cookbooks.  

```
behemphi:~ 10:56:13 > mkdir ~/chef
behemphi:~ 10:56:22 > cd ~/chef
```

This is the area for cookbooks that are not directly related to an application. These cookbooks are common to all (or a large portion) of the infrastructure in general. 

Create the cookbook. Remember that the dash (-) character is not welcome in cookbook names:

```
behemphi:~/chef 11:41:13 > chef generate cookbook sys_base
[SNIP]
behemphi:~/chef 11:41:19 > ls -l sys_base
total 32
-rw-r--r--  1 behemphi  staff    47 Jul 31 11:41 Berksfile
-rw-r--r--  1 behemphi  staff    56 Jul 31 11:41 README.md
-rw-r--r--  1 behemphi  staff  1067 Jul 31 11:41 chefignore
-rw-r--r--  1 behemphi  staff   204 Jul 31 11:41 metadata.rb
drwxr-xr-x  3 behemphi  staff   102 Jul 31 11:41 recipes
drwxr-xr-x  4 behemphi  staff   136 Jul 31 11:41 spec
drwxr-xr-x  3 behemphi  staff   102 Jul 31 11:41 test
```

**What Happened?**

The `chef` client of the ChefDK has created the scaffolding necessary to get started.  

You can see places for recipes, but note that we are missing mention of templates or attributes.  

**--**

The next thing we will need to do is initialize this cookbook for Vagrant.

```
behemphi:~/chef 11:40:23 > cd sys_base
behemphi:~/chef/sys_base 11:41:23 > vagrant init
```

Then we will need to configure Vagrant for the Chef provisioner by overwriting the `Vagrantfile` with this:

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: '10.42.42.11'

  config.vm.provision "chef_zero" do |chef|
    chef.channel = "stable"
    chef.version = "12.10.24"
    chef.nodes_path = "~/chef-base/nodes"
    chef.cookbooks_path = "~/cookbooks"
    
    chef.log_level = "warn"
    chef.run_list = %w{ 
      recipe[sys_base::default]
    }
    
  end

end
{% endhighlight %}

Now issue a `vagrant up` and get a no-op run for the sys_base cookbook.

**What Happened?**

When chef-zero comes up, it looks in the `chef.cookbooks_path` location for cookbooks it the run_list.  In the `chef.run_list` we have specified the `sys_base` cookbook.  It finds that cookbook and executes the default recipe.

Since the default recipe contains no code, nothing happens.

**--**

### An Opinionated Use of the the `default` recipe

When a cookbook runs, the default recipe is always executed unless this behavior is explicitly changed. With this in mind going forward, we will used the `default` recipe to simply provide a list of other recipes in the cookbook to call and the order in which to call them.

Since order matters in Chef, this is a very easy way to simplify the "mental map" one needs to debug configurations when more than one cookbook is in use.

## Installing Packages

Create the file `packages.rb`

```
behemphi:~/cookbooks/sys_base 13:11:26 > touch ~/cookbooks/sys_base/recipes/packages.rb
```

We are going to use the same code from the [third article](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html) :

{% highlight ruby %}
# packages.rb

# The %w is a "white space separated array of literals". It is 
# the preferred syntax for this sort of list.
packages = %w{ dstat gdb git-core lsof screen strace tmux tshark}

# Because packages is an array, it is iterable, so the each method 
# creates a loop. The only line of Chef DSL is the call to `package`
# This is very similar to the notion of `foreach` in other languages.
packages.each do |pkg|
  package pkg
end
{% endhighlight %}

Now `~/cookbooks/sys_base/recipes/default.rb` should read:

{% highlight ruby %}
# default.rb

# First we install packages
# Gotcha: Note we are calling the recipe and _not_ the file, thus the 
# lack of a `.rb` extension.
include_recipe 'sys_base::packages'
{% endhighlight %}

At this point we have achieve a very simple example of using the `default` recipe to "drive" the operation of the cookbook. To see this in action, reprovision the node:

```
behemphi:~/cookbooks/sys_base 12:58:46 > vagrant provision
[SNIP]
==> default: Recipe: sys_base::packages
[SNIP]
```

Note that in this case we did _not_ edit the `Vagrantfile` therefore we need not reload.  This saves considerable time on the reboot.

**What Happened?**

The same thing that happened in the previous article. This time, however we are set up to perform more operations in a readable way. Look for the line in the output that tells you another recipe was called.  The value in reading output structured like this when troubleshooting will become obvious.

**--**

**Pro Tip**

If you follow this one simple rule, you will revered by your colleagues rather than reviled:

"Code is meant to be read not written."  

**--**

## Creating a Group

The next thing on our list to accomplish is to create a group. The chef `group` resource functions like the bash command `groupadd` command.  

As we did with packages lets create a file for this recipe:

```
behemphi:~/cookbooks/sys_base 13:17:36 > touch ~/cookbooks/sys_base/recipes/groups.rb
```

Our goal is to invite developers on to the production systems in a controlled way. This allows them to take on more responsibility for Level 1 incidents.  Developers who much wake up when systems go awry are much more likely to author systems that do not go awry.

{% highlight ruby %}
# groups.rb

# Group for the general developer and Level 1 support
group 'developers' do
  action :create
end

# Group for the trusted individual engineer from each team who 
# elevate privileges for problem solving and Level 2 support
group 'trusted-developers' do
  action :create
end
{% endhighlight %}

If you were to reprovision at this point, nothing would happen.  Why?

Remember that our `chef.run_list` (see the `Vagrantfile`) only contains the default recipe for the cookbook sys_base. So we must add a call to this new recipe in `default.rb`.  It will now look like this:

{% highlight ruby %}
# default.rb

# First we install packages
# Gotcha: Note we are calling the recipe and _not_ the file, thus the 
# lack of a `.rb` extension.
include_recipe 'sys_base::packages'

# Lay down groups for limited user privileges
include_recipe 'sys_base::groups
{% endhighlight %}

Reprovision your node:

```
behemphi:~/cookbooks/sys_base 13:17:49 > vagrant provision
```

**Exercise**

An error occurs.  Carefully read the output and note the following:

* The error occurred at compile time.  This narrows the scope to a ruby or dependency problem.  No code was executed and node remains unchanged.
* The error is a simple string issue. You are told exactly which line and file are causing the issue.  

**--**

Once you have fixed the problem, reprovision your node.  Take note of the fact that the last line tells you 2 of 10 resources were updated.  This is a nice sanity check that it was the two groups we defined in `groups.rb` that were updated. 

### Trust but Verify 

Let's see the change happened with our own eyes as a way to build trust and intuition with the tooling:

```
behemphi:~/cookbooks/sys_base 13:30:07 > vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ cat /etc/group | grep developer
developers:x:1002:
trusted-developers:x:1003:
vagrant@vagrant-ubuntu-trusty-64:~$ exit
```

**What Happened?**

We connected to the guest and check the contents of the `/etc/group` file. Sure enough we find the groups we added, so we exit the guest and return to our work station.

Wouldn't it be great if there was a way to write some simple statements to do this for you every time you provisioned?  That is to verify the actions on the node?  That is what Test Kitchen is for and it will be explained in future articles.

**--**

## Configuring `sshd` 

We are going to dive a bit deeper in configuring `sshd` by manipulating a template for its configuration and using attributes to fill in template values. This is somewhat contrived to show you these important features of a cookbook, so do not reason from this as you can with the use of the default recipe.

### Scaffolding

We will use `chef` from the ChefDK to scaffold:

```
behemphi:~/cookbooks/sys_base 13:57:47 > chef generate template default/sshd_config
[SNIP]
behemphi:~/cookbooks/sys_base 14:00:03 > chef generate attribute sshd
[SNIP]
behemphi:~/cookbooks/sys_base 14:00:03 > chef generate recipe sshd
[SNIP]
```

**What Happened?**

As you can see by the output, in both cases a directory was created and a file. You are now ready to edit the files.

**--**

**Gotcha**

In the case of the template, a subdirectory called `default` was also created.  It is in this specific folder Chef will look for a template first, otherwise you have to explicitly set the file path.  It is almost always a good idea to simply place your templates in the `default` directory.

**--**

**Pro Tip**

The actual file to be written to the node is `/etc/ssh/sshd_config` so it is a good idea to name the template `sshd_config.erb`.  This tells a cookbook reader what the file is expected to be called on disk and makes it easy for them to find things without having to refer to the calling recipe.

The recipe and attribute files are called `sshd` rather than `sshd_config` because they will contain all installation and configuration for the `sshd` process, not just the one file.  

**--**

### Creating an Attribute

Attributes are exceptionally powerful abstractions in there use within a cookbook. However as language constructs, attributes are nothing more than Ruby key-value pairs.

Write this code to `attributes/sshd.rb`:

{% highlight ruby %}
# attributes/sshd.rb

default['kasasa']['sshd']['allowed_users'] = "behemphi sdmouton"
{% endhighlight %}

The power of the attribute abstraction comes from its organization.  So, as such, the above is very deliberately structured:

* Note the namespace "kasasa". This is done to ensure there is no chance of a attribute from another cookbook colliding.  
* Note the namespace to "sshd". This is done to ensure that within the universe of Kasasa cookbooks, the setting is only valid for `sshd`. This is important because "allowed_users" is a common enough setting that collision is likely.
* Note the keyword "default" for the assignment.  In this case, this refers to the lowest level of precedence in the attribute hierarchy.  More on this in the next article. For now, just realize there other keywords for assigning attribute values.
* Note the last key "allow_users" is exactly what the setting is called in `/etc/ssh/sshd_config`.  You might have a better name, but really you don't. Others reading your code that know ssh will be looking for what they are familiar with.  Use the expected name.

Note, consideration of a config setting collisions likelihood is a waste of time. Use namespacing with attributes in a well reasoned way as a standard practice and never loose days of your life tracking down why `sshd` or some other service is not behaving as expected.

If you were to reprovision the node right now, nothing will change.  This attribute is not used.

### Adding the `sshd` Recipe

{% highlight ruby %}
# recipes/sshd.rb

# Explicitly set a variable so we can manipulate the list without concern for 
# changing data at a full node scope.  
allowed_users = node['kasasa']['sshd']['allowed_users']

# Because we are locking down access and Vagrant uses the vagrant user, we need
# To always make sure it is on the list or `vagrant ssh` will not work
allow_users = allow_users + " vagrant"

# Configure sshd
template '/etc/ssh/sshd_config' do
  group 'root'
  mode '0644'
  owner 'root'
  source 'sshd_config.erb'
  variables(
      :ALLOW_USERS => allow_users
  )
end

# Restart the sshd service to load the new configuration
service "ssh" do
  action :restart
end
{% endhighlight %}

Don't forget to update your `recipes/default.rb` file (the default recipe):

{% highlight ruby %}
# default.rb

# First we install packages
# Gotcha: Note we are calling the recipe and _not_ the file, thus the 
# lack of a `.rb` extension.
include_recipe 'sys_base::packages'

# Lay down groups for limited user privileges
include_recipe 'sys_base::groups'

# Lock down sshd to a whitelist
include_recipe 'sys_base::sshd'
{% endhighlight %}

Did you jump the gun and try to reprovision?  If not, try it?  What happened?

**What Happened?**

If you shell in to the box, note that `/etc/ssh/sshd_config` is empty.  This is because Chef did what you told it to. It wrote and empty file. This could have completely locked us out of the machine for all time! 

**--**

We will discuss more about the template resource in the recipe above after we get a sensible template in place.

## Building a Template

Make your `templates/default/sshd_config.rb file look like the below:

```
# *****************************************************************************
# Name:      /etc/ssh/sshd_config
# Purpose:   This is the base sshd configuration file.
# Note:      This file is written by chef-client for each run of the client.
#            If you make changes locally they will be overwritten on the
#            next client run.
#
#            DO NOT ALTER THIS FILE IN ANY WAY UNLESS YOU KNOW PRECISELY WHAT
#            YOU ARE DOING IN RELATION TO SSHD CONFIGURATION AND THE
#            INSTALLATION RECIPES.
# *****************************************************************************
AcceptEnv LANG LC_*
AllowUsers <%= @ALLOW_USERS %>
ChallengeResponseAuthentication no
HostbasedAuthentication no
HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_rsa_key
IgnoreRhosts yes
KeyRegenerationInterval 3600
LoginGraceTime 120
LogLevel INFO
PasswordAuthentication no
PermitEmptyPasswords no
PermitRootLogin no
Port 22
Port 71
PrintLastLog yes
PrintMotd no
Protocol 2
PubkeyAuthentication yes
RhostsRSAAuthentication no
RSAAuthentication yes
ServerKeyBits 768
StrictModes yes
Subsystem sftp /usr/lib/openssh/sftp-server
SyslogFacility AUTH
TCPKeepAlive yes
UsePAM yes
UsePrivilegeSeparation yes
X11DisplayOffset 10
X11Forwarding yes
```

### Deconstructing the Template File

From a Chef point of view there only interesting things here are the substitute parameter value `@ALLOW_USERS`.  We follow the convention that all our attributed keys will be lower case and use underscore for word separation.  In the `sshd_config` file, camel case is used so it is translated accordingly.

In the gray area between Chef configuring the machine and the actual file on disk, note the header block.  It is good practice to warn anyone on the box who might open this file that it should not be directly edited.  This does two things:

* Keeps new folks from creating headaches by "solving a problem" only to have it "unsolved" for them during the next `chef-client` run.
* During live troubleshooting, it helps identify configuration that might not be under Chef control.  Such configuration is likely to drift from machine to machine, thus is an excellent candidate for the cause of inconsistent behavior.

Long and wasteful are the discussions for a well ordered configuration file.  In the end it all amounts to [bike shedding](http://www.bikeshed.org).  There is only one order we all implicitly understand when we want to look something up: alphabetical order. So, rather than implementing an opinion, implement a universal standard. Organize the production configuration file on the node in alphabetical order.  Let your Chef code describe _why_ your settings are that way.  

### Deconstructing the Recipe

The first two statements initialize and identifier called "allowed_users" and assign it the value from our attribute. We ensure that the user `vagrant` is always on the list of `allow_users` so local development is not interrupted. 

For the template resource the group, owner and mode are the typical Linux file permissions. 


{% highlight ruby %}
template '/etc/ssh/sshd_config' do
  group 'root'
  mode '0644'
  owner 'root'
  source 'sshd_config.erb'
  variables(
      :ALLOW_USERS => allow_users
  )
end
{% endhighlight %}

The "template" resource which declares our intention to write a file on the node at `/etc/ssh/sshd_config`.

We can see the source of this file (the actual template) is simply `sshd_config.erb`.  We need not worry about the path of the file because it is in the `templates/default` subdirectory.  

In the template resource we also see a Ruby symbol called `:ALLOW_USERS` to be set with the computed value of allow_users at the top of the recipe.  This is the secret sauce that causes the substitution of the string "behemphi sdmouton vagrant" to be laid down on disk on the node when the `chef-client` runs.

**Pro Tip**

The use of assigned variables in the template resource is something of an antipattern. You should favor writing the attributes directly in the template when ever possible.  This makes for better context in the template and easier reading in the recipes.

**--**

{% highlight ruby %}
service "ssh" do
  action :restart
end
{% endhighlight %}

The service resource controls services through an init-system found on the machine.  This could be something like `upstart` or `systemd`.  Your interface to it is abstracted though.

In the case of our recipe, once we laid down the config file on disk we want to restart the service to pick it up.  Notice that we use the fact that order matters in Chef to make it very simple to see what is happening.

### Converge the Node

Reprovision the node one more time and note the verbosity of the output. In particular:

```
[SNIP]
==> default: +# Name:      /etc/ssh/sshd_config
[SNIP]
==> default:     +AllowUsers behemphi sdmouton vagrant
[SNIP]
```

This shows the line with the desired user names was added to the file as expected.  Shell in just to be sure it works as a sanity check.

Congratulations, you have authored your first non-trivial cookbook.

# Wrapping Up

This article, you have learned about the basic components of a cookbook.  You have written three recipes and used the default recipe to drive them and make the execution order clear.

You have used the output of a `chef-client` run to debug a compile time problem. You have created a template and seen a substitution happen in the Chef output.

# Looking Forward 

In the next article we will look at managing community cookbooks, more sophisticated use of attribute precedence and Berkshelf for managing dependencies between cookbooks.  









