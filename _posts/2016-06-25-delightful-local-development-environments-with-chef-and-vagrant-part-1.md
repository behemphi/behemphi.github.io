---
categories: 
- Vagrant
- Chef
- Development Environment
date: 2016-06-25
layout: post
tags: 
- Vagrant
- Chef
- Development Environment
- DevOps
- OSX
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 1"
---

This is the first post in a series of articles describing how to set up a local development environment for a simple microservice architecture using Vagrant and Chef.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management .  

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

# Table of Contents

1. **[Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html)**
1. [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html) 
1. [ChefDK and the Chef Provisioner](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html)
1. [Launching EC2 Instances with Knife]()
1. [Berkshelf and Cookbook Authoring]()
1. [Using Community Cookbooks]() 
1. [Testing Infrastrucutre with Test Kitchen]()
1. [Team Coding with Chef]()
1. [Multiple Guests with Vagrant]()


# Assumptions

This article is written with the following assumptions about your equipment:

* You are using a Mac with OSX El Capitan or higher.  
* You have super user access to install software

The following assumptions are made about you:
 * You are well versed with the version control system `git` and are familiar with its SaaSy cousin Github.
* You have a passing familiarity with virtual machines, but need no expertise.
* You should be comfortable, though not neccessarily expert, at a *nix command line.

# Goals for this Article

At the end of this article you should have accomplished the following tasks:

* Installed Virtualbox and Vagrant
* Launched a virtual machine on your local workstation
* Accessed the virtual machine
* Demonstrated the ability to write files on the guest from the workstation.

# First Principles

If you are trying to help your manager understand why it is worth a week of your time to set up a disposable development environment, [check out this article](http://devops.com/?p=17060). The executive summary is:

* No more than 30 minutes for a new team member
* No more than 5 minutes to start over
* No more than 5 minutes to move from one team to another
* Runtime changes happen the same way as application changes

For this article it will be important to be precise about the language used:

* Workstation or laptop: The physical hardware device you do your work on.
* Guest: a virtual machine running on your laptop.

# Vagrant-like Powers

Before getting started with Vagrant you will need a "provider."  A provider is nothing more than a virtualization engine for your laptop.  Virtualbox, by Oracle, is free and popular.  VMware Fusion is nominally priced, and has a couple of advantages. 

## Installing Virtualbox

Virtualbox is free and easy so it is where we will start.

1. [Visit the site](https://www.virtualbox.org/wiki/Downloads) and download the software.
1. Double click on the installer and follow the directions.

**What Happened?**

The software is installed in the system path. This can be verified as shown below.

```
behemphi:~ 20:26:06 > vboxmanage -v
5.0.22r108108
```
**--**

**Gotcha!**

You might be thinking, "Hey I am altering my workstation, isn't that what I am trying to avoid?" The answer is, "Yes you are. No you are not."
 
Virtualbox, and Vagrant are tooling _every developer_ in your organization will share. 

**--** 

## Installing Vagrant

Vagrant is "glue code" that allows you to manage the provider (Virtualbox in our case) through Ruby code.  This jibes with the infrastructure-as-code paradigm of the DevOps movement nicely.

1. [Visit the site](https://www.vagrantup.com/downloads.html) and download the software. 
1. Double click on the installer and follow the directions.

**What Happened?**

The software is installed in the system path. This can be verified as shown below.

```
behemphi:~ 20:26:15 > vagrant -v
Vagrant 1.8.4
```

**--**

**Gotcha!** 

Version numbers change frequently for both Virtualbox and Vagrant. Don't sweat the minor updates unless you cannot get past an issue.

**--**

With Virtualbox and Vagrant installed on your workstation, we are ready to fire up a guest and share some code!

## Let's Code Together

Create a directory for a new project we are going to work on together. Initialize it as a git repository.

{% highlight bash %}
behemphi:~ 20:28:30 > cd ~
behemphi:~ 20:28:34 > mkdir kumbaya
behemphi:~ 20:30:27 > cd kumbaya
behemphi:~/kumbaya 20:30:31 > git init
{% endhighlight %}

This should feel just like starting any other software project.

**Gotcha!**

Note the above `git init` command assumes that git is installed. It is further assumed you have a Github account to push code to.

**--**

Now things take a turn to the different. We are going to initialize this project with Vagrant.

{% highlight bash %}
behemphi:~/kumbaya 20:34:44 > vagrant init
A `Vagrantfile` has been placed in this directory ... [snip] 
behemphi:~/kumbaya 20:34:52 > ls -la
total 8
drwxr-xr-x   4 behemphi  staff   136 Jun 25 20:34 .
drwxr-xr-x+ 43 behemphi  staff  1462 Jun 25 20:30 ..
drwxr-xr-x  10 behemphi  staff   340 Jun 25 20:33 .git
-rw-r--r--   1 behemphi  staff  3008 Jun 25 20:34 Vagrantfile
{% endhighlight %}

**What Happened?** 

Vagrant has written a Ruby file to your project in the directory where you issued the command. This file describes how to launch and configure the runtime environment of the application we are going to build.

**--**

**Gotcha!**

Take a few minutes to read the Vagrantfile. It will help you understand what comes next.

**--**

Let's agree we are going to use Ubuntu Trusty for our application in production. Here is all we currently need to acheive that goal:

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

end
{% endhighlight %}

**Gotcha!**

For the purposes of this article, comments from the Vagrantfile are removed. In practice, however, it is strongly recommended to leave them in place and fortify them with your specific situation. _This is the documentation for your development environment!_

**--**

To accomplish our first goal of showing we can start a guest on our workstation we simply issue the `vagrant up` command. It will take a few minutes to download the Trusty image so read ahead while you wait ...

```
behemphi:~/behemphi/kumbaya 20:43:30 > vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/trusty64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'ubuntu/trusty64' is up to date...
==> default: Setting the name of the VM: kumbaya_default_1466905695824_41414
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 4.3.10
    default: VirtualBox Version: 5.0
==> default: Mounting shared folders...
    default: /vagrant => /Users/behemphi/behemphi/kumbaya
```

**What Happened?**

You have a virtual machine running on your workstation. While you are waiting, here is an explanation of some of the more interesting lines:

* default: Importing base box 'ubuntu/trusty64' - This is what is taking so long. It's a big image.
* default: 22 (guest) => 2222 (host) (adapter 1) - Vagrant is setting up some port mapping so you can easily shell (ssh) in to the box. 
* default: SSH address: 127.0.0.1:2222 - If you wanted to you could configure your SSH client to hit this host, but don't. Vagrant has you covered.
* default: Inserting generated public key within guest - Vagrant is setting up your SSH connection so you don't need a password.
* default: The guest additions on this VM [snip] - The base box we downloaded was not built against the version of Virtualbox we are using. Don't sweat it for now.
* default: Mounting shared folders - Vagrant has given us a way to write on the workstation and have these files appear on the guest. 

As noted above, you don't have to set up SSH ports and the like. Vagrant is does it for you. So we can verify it works:

```
behemphi:~/behemphi/kumbaya 20:48:36 > vagrant ssh
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.13.0-65-generic x86_64)

[snip]

0 packages can be updated.
0 updates are security updates.

vagrant@vagrant-ubuntu-trusty-64:~$ 
```

We are now logged in to our newly created guest.

**--**

Let's take our guest for a test drive:

```
behemphi:~/behemphi/kumbaya 20:48:36 > vagrant ssh
```

You are logged in to the guest as the user `vagrant`. Let's try a few things. What user are you?

```
vagrant@vagrant-ubuntu-trusty-64:~$ whoami
vagrant
```

Yep, you are `vagrant`. What are my privileges?

```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo su -l
root@vagrant-ubuntu-trusty-64:~# whoami
root
```

You are now free to completely hose this box. Brag to your friends that you are, in fact, a developer with root! Have fun.

This next one gets a bit confusing so pay careful attention to when you are on the workstation and when you are the guest.

On the guest create a file in `/vagrant`:

```
vagrant@vagrant-ubuntu-trusty-64:~$ cd /vagrant
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ touch created-from-guest
```

On your workstation, list your `kumbaya` directory:

```
behemphi:~/behemphi/kumbaya 21:04:39 > ls -l
total 8
-rw-r--r--  1 behemphi  staff  77 Jun 25 20:43 Vagrantfile
-rw-r--r--  1 behemphi  staff   0 Jun 25 21:03 created-from-guest
behemphi:~/behemphi/kumbaya 21:04:41 >
```

Now on your workstation, create a file:

```
behemphi:~/behemphi/kumbaya 21:04:41 > touch created-on-host
```

On the guest, list the `/vagrant` directory once more:

```
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ ls -l
total 4
-rw-r--r-- 1 vagrant vagrant  0 Jun 26 02:03 created-from-guest
-rw-r--r-- 1 vagrant vagrant  0 Jun 26 02:05 created-on-host
-rw-r--r-- 1 vagrant vagrant 77 Jun 26 01:43 Vagrantfile
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ 
```

This simple exercise shows that you can use all the tools you love on your workstation (host) while having the code available on the guest in real time.  If you are working in a language such as PHP, Python or Ruby, you can essentially excercise your code on a model of the production platform in real time.

# Wrapping Up 

You have Installed Virtualbox and Vagrant and shown them to work by launchign a guest on your workstation with the `vagrant up` command. You accessed the guest by using Vagrant's handing port mappings with `vagrant ssh`. 

In the advanced stage you showed that you could write a file on your workstation and have that file immediately appear on the guest.  This means you can write code on the workstation and use all your favorite tools while the files will also be available on the guest.  The implication is this will improve SDLC workflow.

# Looking Foward

In the next article we will use the vagrant SHELL provisioner to configure the guest and run a small application.  
