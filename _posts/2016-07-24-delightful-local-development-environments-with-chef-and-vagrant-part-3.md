---
categories: 
- Vagrant
- Chef
- Development Environment
date: 2016-07-24
layout: post
tags: 
- Vagrant
- Chef
- Development Environment
- DevOps
- OSX
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 3"
---

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management with Chef . 

# Table of Contents

1. [Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html) 
1. [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html)
1. **[ChefDK and the Chef Provisioner](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html)**
1. [Launching EC2 Instances with Knife](/vagrant/chef/development%20environment/2016/07/28/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)
1. [Cookbook Authoring](/vagrant/chef/development%20environment/2016/07/30/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)
1. [Using Community Cookbooks and Berkshelf](/vagrant/chef/development%20environment/2016/08/01/delightful-local-development-environments-with-chef-and-vagrant-part-6.html)
1. [Testing Infrastrucutre with Test Kitchen]()
1. [Team Coding with Chef]()
1. [Multiple Guests with Vagrant]()

# Goals for this Article

At the end of this article you should be able to:

* Install ChefDK
* Set up Vagrant to configure guests with Chef
* Create a cookbook 
* Configure a guest with Chef
* Understand some important Chef concepts and lingo

# First Principles

For this article and those going foward make the following distinctions:

* Chef is the company that makes the Chef configuration managment software (as well as Inspec and Habitat)
* `chef client` (note the case) is the tooling installed on your guest that executes your cookbook
* The "chef server" is a server we can think of as "somewhere else" for the time being. It stores useful information about each node and makes your infrastructure queryable. 
* "Chef Zero" is a light weight in-memory version of the chef server used only for development and testing of cookbook code.  It comes in ChefDK and has a command line interface `chef-zero`
* ChefDK (Chef Development Kit) is a package of Ruby tools like Chef Zero and `chef-client` that you download as a single developer toolkit.

This article is necessarily heavy on Chef concepts and definitions.  The above are key, however more will be introduced below.  Take the time to think through each as it will save you hours of frustration troubleshooting.

# Installing ChefDK  

As before, be sure to check your status and ensure you have no guest running:

{% highlight bash %}
behemphi:~/kumbaya 21:38:03 > vagrant status
[SNIP]
behemphi:~/kumbaya 21:38:16 > vagrant destroy
==> default: VM not created. Moving on...
{% endhighlight %}

**What Happened?**

You checked the status of the guest for the Kumbaya project. Just to be sure no guest is running, or in an inconsistent state, you issued a destroy command. This command completely removed the guest from your workstation.

Again, you have completely trashed the run time environment of the application but, by now, you should not be worried. You know you can get it all back with a `vagrant up`!

**--**

The problem with the SHELL provisioner is that it is not [idempotent](https://en.wikipedia.org/wiki/Idempotence).  In short, we don't have an absolute idea of what will happen were we to run the shell script again.  While we could figure it out, Chef handles the reapplicaiton of the configuraiton in a reliable and effiecient way.  

## Installing the ChefDK

ChefDK is provided as an omnibus installation. This will be important when you want to manipulate Ruby gems and perform other, more advanced, operations.  

The important thing for getting started is that installing ChefDK does not depend on system Ruby.  Installation is, therefore, a snap:

1. [Visit the ChefDK download page](https://downloads.chef.io/chef-dk/mac/) and choose the matching OS download.
1. Open the dowloaded disk image, double click the installer and follow the directions
1. Unmount/Eject the disk image.

**What Happened?**

Ruby, the Chef ecosystem, BerkShelf and Test Kitchen were all installed in their own directory with their own ruby.  This can be verified:

```
behemphi:~/kumbaya 12:39:05 > chef -v
Chef Development Kit Version: 0.16.28
chef-client version: 12.12.15
delivery version: master (921828facad8a8bbbd767368bfc72f19bd30e7bd)
berks version: 4.3.5
kitchen version: 1.10.2
```

**--**

## Configuring Vagrant

In your version of the kumbaya project, replace your vagrant file with the following:

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: '10.42.42.10'

  config.vm.provision "chef_zero" do |chef|
    chef.channel = "stable"
    chef.version = "12.10.24"
    chef.nodes_path = "~/chef-base/nodes"
    # chef.cookbooks_path = "cookbooks"
    
    # chef.log_level = "warn"
    # chef.run_list = %w{ app_kumbaya::default }
    
  end

end
{% endhighlight %}

As you can see, the `config.vm.provsion` section has changed dramatically.  Here is a tour of what we are doing.

* [chef.channel](https://www.vagrantup.com/docs/provisioning/chef_common.html) - Allows us to choose the dowload area for the version of the Chef Client that will be running.  We choose "stable" because we don't want to be chasing down odd ball issues while just getting started.
* [chef.version](https://www.vagrantup.com/docs/provisioning/chef_common.html) - While not strictly required, it is a very good idea to pin your chef-client version. Chef-client and vagrant can get out of sync and bugs can happen.  It is better to upgrade often but _intentionally_.
* [chef.nodes_path](https://www.vagrantup.com/docs/provisioning/chef_zero.html) - When the chef-client runs, it will store the node state on disk so that it can mimic state.  This is necessary because Chef Zero does not live past the end of the vagrant run.  This is a housekeeping location on your workstation and should _not_ be part of a repo unless you `.gitignore` it. 
* [chef.cookbooks_path](https://www.vagrantup.com/docs/provisioning/chef_zero.html) - The location to find the cookbooks on your workstation. This is relative to the location of the Vagrantfile and must be named `cookbooks`. We are commenting it out to use the default behavior.

The next two values will be used later in the article, but are commented out for the time being.

* [chef.log_level](https://www.vagrantup.com/docs/provisioning/chef_apply.html) - The verbosity of logging from `chef-client` is set here.  We will set this after witnessing the verbosity of the client.
* chef.run_list - An ordered list of cookbooks to be applied to a guest.

We can now test the installation of ChefDK more fully.

### Chef No-Op Run

From the `chef.nodes_path` above we need to create a directory:

```
behemphi:~/kumbaya 13:43:27 > mkdir ~/chef-base/nodes
```

Launch a guest virtual machine by:

```
behemphi:~/kumbaya 13:43:45 > vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
[SNIP]
==> default: [2016-07-24T18:45:17+00:00] INFO: Started chef-zero at chefzero://localhost:8889 with repository at 
[SNIP]
Crazy amounts of output
[SNIP]
==> default: --- END RESPONSE ---
==> default: [2016-07-24T18:45:19+00:00] INFO: Chef Run complete in 0.072628642 seconds
==> default: 
==> default: Running handlers:
==> default: [2016-07-24T18:45:19+00:00] INFO: Running report handlers
==> default: Running handlers complete
==> default: [2016-07-24T18:45:19+00:00] INFO: Report handlers complete
==> default: Chef Client finished, 0/0 resources updated in 01 seconds
```

**What Happened?**

Vagrant brought up the machine in the way we are used to seeing.  

Chef created a node object. This is a massive amount of JSON cataloging everything it knows or expects to know about the node.  It writes this to the `chef.nodes_path	` on your workstation.

Finally, at the end you can see that the chef-client run completes successfully and that no resources were changed.

**--**

In the parlance of Chef, a "node" is the object to be configured.  It is generally a virtual machine, but could also be a physical machine, a Juniper switch or even a Docker container. In the case of this article a "node" can be taken to mean the Vagrant guest.

A "resource" is a unit of configuration.  For example, installing a package from the OS repository, or creating a directory with the right permissions.  More on this later.

While it is great to see all of the node information the first time, it will quickly be difficult to iterate in development.  Comment out the `chef.log_level` line of the `Vagrantfile` and do the following:

```
behemphi:~/kumbaya 13:48:48 > vagrant reload --provision
```

**Gotcha!**

Because we just changed the `Vagrantfile` we _must_ issue the command `vagrant reload`.  This command will restart the guest and apply any changes from the `Vagrantfile`.  

**--**

## Scaffolding a Cookbook

We are going to create a cookbook in our Kumbaya Python application to configure the host.  

`chef` is CLI tooling for doing scaffolding tasks like setting up a cookbook, etc. Before doing so, type `chef` at the commandline and take the time to read through the output.  

The `chef_zero` provisioner has an opinionated way of looking for your cookbook.  As such, you will need to create a top level directory in your kumbaya project.

```
behemphi:~/kumbaya 17:16:01 > mkdir ~/kumbaya/cookbooks
behemphi:~/kumbaya 17:16:04 > cd ~/kumbaya/cookbooks
behemphi:~/kumbaya/cookbooks 17:16:15 > chef generate cookbook app_kumbaya
Generating cookbook app_kumbaya
- Ensuring correct cookbook file content
- Ensuring delivery configuration
- Ensuring correct delivery build cookbook content

Your cookbook is ready. Type `cd app_kumbaya` to enter it.
[SNIP]
```

**What Happend?**

The Chef CLI tool created a number of directories and files for us.

```
behemphi:~/kumbaya 17:16:24 > ls -la app_kumbaya/
total 48
drwxr-xr-x  12 behemphi  staff   408 Jul 24 17:16 .
drwxr-xr-x  11 behemphi  staff   374 Jul 24 17:16 ..
drwxr-xr-x   5 behemphi  staff   170 Jul 24 17:16 .delivery
-rw-r--r--   1 behemphi  staff   126 Jul 24 17:16 .gitignore
-rw-r--r--   1 behemphi  staff   236 Jul 24 17:16 .kitchen.yml
-rw-r--r--   1 behemphi  staff    47 Jul 24 17:16 Berksfile
-rw-r--r--   1 behemphi  staff    59 Jul 24 17:16 README.md
-rw-r--r--   1 behemphi  staff  1067 Jul 24 17:16 chefignore
-rw-r--r--   1 behemphi  staff   213 Jul 24 17:16 metadata.rb
drwxr-xr-x   3 behemphi  staff   102 Jul 24 17:16 recipes
drwxr-xr-x   4 behemphi  staff   136 Jul 24 17:16 spec
drwxr-xr-x   3 behemphi  staff   102 Jul 24 17:16 test
```

**--**

**Gotcha!**

Chef does not like the dash character in cookbook names.  Additionally, Ruby does not allow the dash character as part of identifier names.  This is why the cookbook directory, unlike others, will use the underscore character for word seperation.

Avoid the use of the dash character in identifiers and file names.

**--**

The files and directories we are interested in for this article are:

* metadata.rb - Global information about the cookbook as  single artifact.  Take a moment to fill in your own information.  We will become very interested in the `version` in future articles as we manage deployments.
* recipes/default.rb - Recipes contain code that configures your guest.

## Authoring a Cookbook 

Open the file `~/kumbaya/app_kumbaya/recipes/default.rb and add the following code.  Ruby explanations are provide as comments

{% highlight ruby %}
# default.rb

# The %w is a "white space seperated array of literals". It is 
# the preferred syntax for this sort of list.
packages = %w{ dstat gdb git-core lsof screen strace tmux tshark}

# Because packages is an array, it is iteralbe, so the each method 
# creates a loop. The only line of Chef DSL is the calle to `package`
# This is very similar to the notion of `foreach` in other languages.
packages.each do |pkg|
  package pkg
end
{% endhighlight %}

A powerful thing to note here is the DSL resource `package`.  It does not take an operating system argument. Instead, it will install the named package be it on CentOS, Debian, FreeBSD or Windows.  It is the job of a "provider" in the parlance of Chef (not Vagrant) to handle the implementation details for each OS. 

Finally we will add the run list to the guest node.  A "run list" is an _ordered list_ of cookbooks to be run by the `chef-client` on the guest node. Remove the comment from the `chef.run_list`  line of the vagrant file.

Instead of destroying our existing guest and recreating it, lets be a bit less heavy handed and save some time. Because we edited the `Vagrantfile` we need to reload. We then want the provisioner to run.  

```
behemphi:~/kumbaya 19:28:48 > vagrant reload --provision
[SNIP]
==> default: Starting Chef Client, version 12.10.24
==> default: resolving cookbooks for run list: ["app_kumbaya::default"]
==> default: Synchronizing Cookbooks:
==> default:   
==> default: - app_kumbaya (0.1.0)
[SNIP]
==> default: Running handlers:
==> default: Running handlers complete
==> default: 
==> default: Chef Client finished, 4/8 resources updated in 46 seconds
```

**What Happened?**

In the first block we can see the chef-client start up.  Notice that the wait for the client to be installed did not happen because we only restarted the machine.  The run list is defined as `app_kumbaya::default` just as we said. The client checks itself to see if it has the right version of the cookbook (0.1.0 in our case) and then begins to apply the recipe. Otherwise it would get the right cookbook version from the Chef Server (`chef-zero` in the case of our development environment).

On your workstation scroll back up and carefully inspect the output. You will see each package being installed.

Finally Chef tells us that 4/8 resources were updated.  The "missing" 4 resources, in this case, are due to the fact that some packages were already present and there was no need to apply them again.  

**--**

Let's verify that we have what we think we should:

```
behemphi:~/kumbaya 20:31:46 > vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ dpkg --list | grep lsof
ii  lsof                             4.86+dfsg-1ubuntu2                  amd64        Utility to list open files
vagrant@vagrant-ubuntu-trusty-64:~$ exit
behemphi:~/kumbaya 20:32:40 >
```

**What Happend?**

We used vagrant to conveniently shell into the guest and checked to see if the package `lsof` was present. We then exited from the guest back to the workstation.

**--**

Congratulations. You have converged your first node with Chef!

## Anatomy of a `chef-client` Run

Now that you have your very own Chef configured Vagrant guest it is time to consolidate Chef concepts and definitions.

What you have done is "converged a node."  This means that the `chef-client` ran and brought your guest (the node) into the desired state.  It is assumed that the chef-client will run every so often and re-converge the node as needed.

A ["resource"](https://docs.chef.io/resources.html) is a declaration of state about a discrete aspect of the system.  In the example above this was a OS package. It could also be a directory and its permissions along with a configuration file. Resources are agnostic to the OS on the node. In our case, since we are using Ubuntu, the "provider" called by Chef is `apt`. 

When the chef-client runs, it has two phases:

* compile
* converge

During the compile phase,  cookbooks are evaluated as for their correctness as Ruby Code.  The resources declared in the recipes are added to a collection that will be applied to the node.

After the compile phase, the converge phase is entered.  The resource collection is applied to the node as a set of providers.  A provider is the OS specific implementation of the resource.  Chef checks the OS type and version via a gem called Ohai. 

Being aware of where problems occur (compile vs. converge) will save you loads of time tracking down problems as you write your own cookbooks.

Understand that the order you write your resources in a recipe is the order they are applied to a node.  This fact makes, couple with the `chef-client`'s verbosity, helps to track down problems as well.

# Wrapping Up

You have installed the ChefDK on your workstation and wired it in to Vagrant via a new `Vagrantfile`.  

Using the `chef` CLI you created the scaffolding of your first cookbook.  You then used the package resource to install a few OS packages and verified they are present on the guest.

With that experience, you then started down the road of building up your understanding of Chef concepts and terms like "converge" and "resource." 

# Looking Forward

In the next article we will take a short detour to launch a node into Amazon Web Services EC2 with Chef only.  This will more allow you to experiment with delivering Chef configured infrastructure as you learn more cookbooks authoring and Vagrant.