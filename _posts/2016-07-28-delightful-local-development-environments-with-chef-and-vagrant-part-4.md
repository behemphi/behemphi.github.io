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
- Knife
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 4"
---

This is the second post in a series of article describing how to set up a local development environment for a simple microservice architecture using Vagrant and Chef.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management .  

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

# Table of Contents

1. [Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html) 
1. [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html)
1. [ChefDK and the Chef Provisioner](/vagrant/chef/development%20environment/2016/07/24/delightful-local-development-environments-with-chef-and-vagrant-part-3.html)
1. **[Launching EC2 Instances with Knife](/vagrant/chef/development%20environment/2016/07/28/delightful-local-development-environments-with-chef-and-vagrant-part-4.html)**
1. [Using Community Cookbooks]() 
1. [Testing Infrastrucutre with Test Kitchen]()
1. [Team Coding with Chef]()
1. [Multiple Guests with Vagrant]()

# Goals for this Article

At the end of this article you should be able to:

* Create a Hosted Chef server account
* Install and configure the `knife ec2` plugin
* Launch and EC2 instance with `knife`

# First Principles

For this article it is important to make the following distinctions:

* "Hosted Chef" is a SaaS offering of the chef server.  It provides a persistent way to store information about your infrastructure.  In the last article we used Chef Zero as a mock Hosted Chef
* The  command line interface `knife` is used to work with the Chef Server. It is the primary tool for working with your Chef based infrastructure
* A `knife` plugin allows `knife` to work with various cloud providers or perform other tasks.  `knife` is an extremely flexible tool.

This article is not about development environments at all. Rather it is about what you do when your development environment has produced something to be used in another environment (e.g.  UAT or Production). 

# Cutting to the Chase

## Creating a Hosted Chef Account

Visit the [Hosted Chef Signup Page](https://manage.chef.io/signup) and fill out the form. You will need to do an email verification.

**Gotcha!**

You should carefully consider what you would like to do in terms of your initial set up.  If, for example, you have separate AWS accounts for DEV and PROD environments, then mabye you want your company name to be myCompany-dev.  

Either way, it is recommended you admit, "I don't know enough right now" and use a throw away name with the idea you will start from scratch when you know more.

**--**

After email verification you are presented with the ability to create a New Organization.  

In your Hosted Chef company, you can create multiple organizations.  Lets create one for sandbox activities of learning. Click the New Org Create button and choose a descriptive name.  

**Pro Tip**

Don't bother mucking around in the Hosted Chef UI too much. While it is a nice user experience, you will ultimately spend little to no time there.  If you can do it in the UI, you can do it with `knife`.  More importantly, you will _want_ to do it with `knife`

**--**

## Generating the Knife Config

Chef's `knife` is a command line client for the Chef Server. It was installed as part of the omnibus install of Chef in the last article.  Chef makes it very easy to get started with `knife`. 

Click the "Download Starter Kit: button. A directory comes down to your workstation.  Check that knife works in a quick an dirty way:

```
behemphi:~ 20:11:13 > knife client list
WARNING: No knife configuration file found
WARN: Failed to read the private key /etc/chef/client.pem: #<Errno::ENOENT: No such file or directory @ rb_sysopen - /etc/chef/client.pem>
ERROR: Your private key could not be loaded from /etc/chef/client.pem
Check your configuration file and ensure that your private key is readable
behemphi:~ 20:11:17 > cd ~/Downloads/chef-repo/
behemphi:~/Downloads/chef-repo 20:11:30 > knife client list
bancvue-sandbox-validator
```

**What Happened?**

In my home directory, I attempted to invoke `knife` for list of clients.   I get a warning saying the tool is not configure.

I then changed directory to the download and invoke `knife` again. This time I get a single client.  This is the security object all my requests to the chef server will be validated against.

More on clients later.

**--**

This is pretty cludgy as we will want to run knife in any context on our workstation. To do this we will take the `.chef` directory from the download and copy it into our home directory.

```
behemphi:~ 20:42:01 > cp -r ~/Downloads/chef-repo/.chef ~/
behemphi:~ 20:44:13 > knife client list
bancvue-sandbox-validator
```

**What Happened?**

Chef tools look for the `~/.chef` directory and load that configuraiton by default.  So when `knife` is invoked, it checks this directory and uses this configuration.

**--**

## Installing the `knife-ec2` Plugin

By default `knife` is not set up to launch Amazon EC2 instances.  We need only install a plugin for this.  

```behemphi:~/kumbaya 20:51:21 > sudo chef gem install knife-ec2
Password:
Fetching: fog-1.29.0.gem (100%)
Successfully installed fog-1.29.0
Fetching: knife-ec2-0.12.0.gem (100%)
Successfully installed knife-ec2-0.12.0
2 gems installed
behemphi:~/kumbaya 20:54:23 > knife ec2
FATAL: Cannot find subcommand for: 'ec2'
Available ec2 subcommands: (for details, knife SUB-COMMAND --help)

** EC2 COMMANDS **
knife ec2 amis ubuntu DISTRO [TYPE] (options)
knife ec2 flavor list (options)
knife ec2 server create (options)
knife ec2 server delete SERVER [SERVER] (options)
knife ec2 server list (options)
```

**What Happened?**

We used `sudo` and `chef gem` to install the `knife-ec2` gem.  It is important to realize the `chef gem` command invodes the omnibus `chef` client and that client installs the plugin on your behalf to the omnibus area.  No changes have been made to your system ruby.

We then verified that the gem will function. Note that while the command fails it gives you a nice set of usage hints.  This is typical of all Chef CLI tools.

**--**

Open your `~/.bash_profile` and add the following lines to the end:

```
# AWS Credentials
export AWS_ACCESS_KEY_ID=AKIAICXXXCEOTKCT452X
export AWS_SECRET_ACCESS_KEY=zrpQVAafBq5BKj3gI4k4skxXX/oBWP4WE7VlV7Xx
ssh-add ~/.ssh/sandbox-bancvue.pem
```
The `.pem` file is your private key in EC2 that allows you to shell in to an instance.  If you this is not clear, check out [the EC2 docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws).

Open your `~/.chef/knife.rb` file and add the following lines: 

```
# AWS Credentials
knife[:aws_access_key_id] = "AKIAICXXXCEOTKCT452X" 
knife[:aws_secret_access_key] = "zrpQVAafBq5BKj3gI4k4skxXX/oBWP4WE7VlV7Xx"
```

Now source the your `~/.bash_profile`:

```
behemphi:~ 21:11:11 > . ~/.bash_profile
```

**Gotcha!**

If you often have more than one terminal open, close all but one or remember to source `~/.bash_profile` in every window.  

**--**

Test the set up by listing the different types of instanes you can launch:

```
knife ec2 flavor list
ID         Name                  Architecture  RAM      Disk      Cores
c1.medium  High-CPU Medium       32-bit        1740.8   350 GB    5
c1.xlarge  High-CPU Extra Large  64-bit        7168     1690 GB   20
[SNIP]
```

**What Happened?**

While it is obvious you got a list of the different types of instances (nodes) you can launch, more important is that you got a response from the AWS API.  This means you are ready to get launching nodes.

**--**

## Launching an EC2 Instance with Knife

Consider but do not try to run the command:

```
behemphi:~ 22:01:32 > knife ec2 server create \
  --node-name first-node \
  --image ami-4403ea24 \
  --security-group-ids sg-e57b2482 \
  --ssh-key sandbox-sysops-boyd.hemphill \
  --region us-west-2 \
  --flavor t2.small
  
Instance ID: i-0a3c49d7
Flavor: t2.small
Image: ami-4403ea24
Region: us-west-2
Availability Zone: us-west-2c
Security Group Ids: sg-e57b2482
Tags: Name: first-node
SSH Key: sandbox-sysops-boyd.hemphill
[SNIP]
```

First let's decompose it:

* `--node-name` - This is the name of the node as Chef will recognize it. It will also be the instance name in EC2.
* `--image` - The AMI to launch as a node.  It is important to be sure the image is availabe in the region (below) or you will get an error.
* `--security-group-ids` - The security group to launch the image into. This cannot be changed after launch. So, be sure you can reach the host from where you would like to test the connection.  You will have to use your own security group. The one in the command above is account specific.
* `--ssh-key` - The key pair name. You can find this information in the AWS EC2 console by clicking "Key Pairs" under Network & Security, and choosing the name of the pair you have access to.
* `--region` - The region (us-west) and the availability zone (2) you would like to launch the instance in.
* `--flavor` - The flavor of the instance. 

As you can see, many of these settings are specific to your AWS environment.  So take your time, play around and troubleshoot.  

# Wrapping Up

You created your first Hosted Chef account and organization. This will allow you to begin to realize the power of Chef beyond just configuring a single node in your infrastructure.

You then configure the CLI tool `knife` so that it can talk to the Hosted Chef server.  You performed a simple query of objects in the Chef Server.

Next you installed the `knife-ec2` plugin so that we could directly manipulate both our Chef Server and our EC2 environment at the same time.  

You then launched an EC2 instance with knife.  

# Looking Forward

In the next article we will author a simple cookbook and promote it to the Chef Server.  We will then launch a Chef configured EC2 instance while learning more about the various artifacts that make up the Chef ecosystem.




