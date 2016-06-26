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
title:  "Delightful Local Development Environments with Chef and Vagrant"
---

# Delightful Local Development Environments

This will be a series of three articles describing how to set up a local development environment for a simple microservice architecture using Vagrant, Chef and Python.  

This the the first article and it will cover the basics of the disposable development environment paradigm and walk you through the installation and use of [Vagrant](https://www.vagrantup.com).

# Assumptions

This article assumes you are well versed with the version control system `git` and have some familiarity with its SaaSy cousin Github.

You should have a passing familiarity with virtual machines, but need no expertise.

You should be comfortable, though not neccessarily expert, at a *nix command line.

## Start with Why

An important goal for a team practicing DevOps should be developer portability. If you, as a dev, cannot pick up your laptop and move to another team down the hall and build the code base in less than an hour, then your company is suffering.

The concept here is that of a "disposable development environment." It's advantages are:

* Developer Portability
* Environment Parity
* Ease of Collaboration
* Innovation Acceleration

It won't be obvious until you have completed this article, but the use of Vagrant enables nothing short of being able to pick up your laptop, walk down the hall and issue the following commands:

* `git clone` 
* `vagrant up`

At this point you should be able to build the system you are working on.  You will note the lack of changing settings from the environment you were working in. You will notice the lack of sitting with another person from the new team in a 5 hour "try this" session because the wiki is out of date.



Conceptually, using one or more virtual machines to model your application 
run time in development divorces the runtime from the writing of code. 



So the 
If you are already familiar with the reasons to use a tool like Vagrant, you can jump down to the next section. I suspect you will pick up a few things here though that might help you sell your peers, supervisor, etc on the change.

## Disposable Environments mean Faster Innovation



## Vagrant like Powers

Before getting started with Vagrant you will need a "provider."  A provider is nothing more than a virtualization engine for your laptop.  Virtualbox, by Oracle, is free and popular.  VMware Fusion is nominally priced, and has a couple of advantages. 

### Installing Virtualbox

Before installing Vagrant you will need a provider, so let's install Virtualbox since it is free and easy.

1. [Visit the site](https://www.virtualbox.org/wiki/Downloads) and download the software.
1. Double click on the installer and follow the directions.
1. Open a terminal and run the below command to verify the install:

```
behemphi:~ 20:26:06 > vboxmanage -v
5.0.22r108108
```

You will be show the version information. This verifies the software is installed.

Gotcha: You might be thinking, "Hey I am altering my workstation, isn't that what I am trying to avoid?" The answer is, "Yes you are. No you are not. This is tooling _everyone_ in your organization will share. A platform if you will."

### Installing Vagrant

We are now ready to install Vagrant.

1. [Visit the site](https://www.vagrantup.com/downloads.html) and download the software. 
1. Double click on the installer and follow the directions.
1. Open a terminal and run the below command to verify the install:

```
behemphi:~ 20:26:15 > vagrant -v
Vagrant 1.8.4
```

**Gotcha:** 

Version numbers change frequently for both Virtualbox and Vagrant. Don't sweat the minor updates.

--

### Let's Code Together

Create a directory for a new project we are going to work on together. Initialize it as a git repository.

{% highlight bash %}
behemphi:~ 20:28:34 > mkdir kumbaya
behemphi:~ 20:30:27 > cd kumbaya
behemphi:~/kumbaya 20:30:31 > git init
{% endhighlight %}

Now initialize it as a project Vagrant is going to support the runtime for. It will look something like this:

{% highlight bash %}
behemphi:~/kumbaya 20:34:44 > vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
behemphi:~/kumbaya 20:34:52 > ls -la
total 8
drwxr-xr-x   4 behemphi  staff   136 Jun 25 20:34 .
drwxr-xr-x+ 43 behemphi  staff  1462 Jun 25 20:30 ..
drwxr-xr-x  10 behemphi  staff   340 Jun 25 20:33 .git
-rw-r--r--   1 behemphi  staff  3008 Jun 25 20:34 Vagrantfile
{% endhighlight %}

**What Happened:** 

Very little actually. Vagrant has written a Ruby file to your project in the directory where you issued the command.

We are now ready to configure Vagrant.

--

**Pause:**

Take a few minutes to read the Vagrantfile. It will help you understand what comes next.

--

Let's agree we are going to use Ubuntu Trusty for our application in production.  Given that fact, we set the goal to bring up a virtual server in on our workstation.

Here is all we currently need to acheive that goal:

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

end
{% endhighlight %}

**Gotcha:**

For the purposes of this article, comments from the Vagrantfile are removed. In practice, however, it is strongly recommended to leave them in place and fortify them with your specific situation. _This is the documentation for your development environment!_

--

So let's accomplish our first goal of showing we can start a virtual machien on our personal workstation that represents the base machine we will use in production.

For the first run it will take a few minutes, so read ahead ...


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

**What Happened:**

Ultimately you have a virtual machine running on your workstation. But while you are waiting, here is an explanation of some of the more interesting lines:

* default: Importing base box 'ubuntu/trusty64' - This is what is taking so long. It's a big image.
* default: 22 (guest) => 2222 (host) (adapter 1) - Vagrant is setting up some port mapping so you can easily shell (ssh) in to the box. 
* default: SSH address: 127.0.0.1:2222 - If you wanted to you could configure your SSH client to hit this host, but don't. Vagrant has you covered.
* default: Inserting generated public key within guest - Vagrant is setting up your SSH connection so you don't need a password.
* default: The guest additions on this VM <snip> - The base box we downloaded was not built against the version of Virtualbox we are using. Don't sweat it for now.
* default: Mounting shared folders - Vagrant has given us a way to write locally and save on the guest.

--

**Pro Tip:**

Your workstation is the "host" for this excersise.  The virtual machine you just started is a "guest".  These terms will be used going forward.

--

Let's take our guest for a test drive:

```
behemphi:~/behemphi/kumbaya 20:48:36 > vagrant ssh
Welcome to Ubuntu 14.04.3 LTS (GNU/Linux 3.13.0-65-generic x86_64)

<snip>

0 packages can be updated.
0 updates are security updates.


vagrant@vagrant-ubuntu-trusty-64:~$ 
```

You are logged in to the guest as the user `vagrant`. Let's try a few things:

What user are you?

```
vagrant@vagrant-ubuntu-trusty-64:~$ whoami
vagrant
```

Yep, you are `vagrant`.

What are my privileges?

```
vagrant@vagrant-ubuntu-trusty-64:~$ sudo su -l
root@vagrant-ubuntu-trusty-64:~# whoami
root
```

You are now free to completely hose this box. Brag to your friends that you are, in fact, a developer with root! Have fun.

This next one gets a bit confusing so pay careful attention to when you are on the host and when you are the guest.

On the guest create a file in `/vagrant`:

```
vagrant@vagrant-ubuntu-trusty-64:~$ cd /vagrant
vagrant@vagrant-ubuntu-trusty-64:/vagrant$ touch created-from-guest
```

On your host, list your `kumbaya` directory:


```
behemphi:~/behemphi/kumbaya 21:04:39 > ls -l
total 8
-rw-r--r--  1 behemphi  staff  77 Jun 25 20:43 Vagrantfile
-rw-r--r--  1 behemphi  staff   0 Jun 25 21:03 created-from-guest
behemphi:~/behemphi/kumbaya 21:04:41 >
```

Now on your host, create a file:

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

In reality the above is quite a jump in implementation. Nobody uses a base Ubuntu install. We have to install our runtime, monitoring agents, logging paradigm, etc.  

### Digestion

Before we get to configuring our guest, let's justify the work we just did and are about to do.

If you were to share this repo with me, I could easily clone it, start the guest and add value.  Consider this on a team of five developers. An update to the runtime environment is a simple pull.  The guest can be destroyed at any time without losing a single line of code!

Wait, that is _not_ true.  Right now we would have to log in and configure the machine at the CLI. 

Let's make it true.

### Sharing Configuration

Since the guest we have is one we don't want, let's get rid of it:

```
behemphi:~/behemphi/kumbaya 21:42:53 > vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```

**What Happened:**

Vagrant passed the name of the guest to Virtualbox along with the command to shutdown and destroy all associated files.  The machine is gone.  Any data assoicated with it that lived on your workstation was unaffected.

--

It may have felt a bit odd to through work away without so much as a thought, but this is the beginning of another powerful concept: disposability. You can try whatever you like with the development environment and if you hose it, you simply destroy it and try again. Because the code describing the environment is part of your project, you can always revert to a working state.  

What we need is a way for the new guest to be configured each time we `vagrant up`. Vagrant exposes this functionality to us with the concept of a provisioner.

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.provision "shell", inline: <<-SHELL
    
    # Apply the latest OS updates
    sudo apt-get update
    sudo apt-get upgrade

    # Install Python goodies
    sudo apt-get install --assume-yes python-pip
    sudo pip  install flask
    sudo pip  install flask-login
    sudo pip  install flask-openid
    sudo pip  install flask-mail
    sudo pip  install flask-sqlalchemy
    sudo pip  install sqlalchemy-migrate
    sudo pip  install flask-whooshalchemy
    sudo pip  install flask-wtf
    sudo pip  install flask-babel
    sudo pip  install guess_language
    sudo pip  install flipflop
    sudo pip  install coverage

  SHELL

end
{% endhighlight %}

Paste the above into your `kumbaya/Vagrantfile` and issue a `vagrant up` command. This time things take much longer and there 100's of lines of green output. 

**What Happened:**

After Vagrant launched your guest, it executed the shell commands found in the provisioner section of the file. 

**Gotcha:**

This is a powerful demonstration, but its a very blunt instrument. You should not attempt to re-run the provisioner because there is no guarentee that any shell command will have the expected effect a second time through. Better to destroy the machine and `vagrant up` again for now.

In a future article, Chef will replace the generic shell provider to address this very problem.

--

It should now be clear that you are able to install a run time - Python's Flask in this case - and work with anyone on your team to contribute to the run time.  Realize too, this exact shell script can be used to configure the production application servers.  Envrionment parity for the win!

### Hello Vagrant World

Let's create a simple "Hello World" application with Flask. First we need to lay down three python files:

`kumbaya/run.py`
{% highlight python %}
#!/usr/bin/python
from app import app
app.run(host="0.0.0.0")
{% endhighlight %}

`kumbaya/app/__init__.py`
{% highlight python %}
from flask import Flask

app = Flask(__name__)
from app import views
{% endhighlight %}

`kumbaya/views.py`
{% highlight python %}
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
{% endhighlight %}

If you were running this on your workstation, you would have to visit the address `127.0.0.1:5000`, but note in the new line of the Vagrantfile below we have explicitly assigned an address to the guest of 10.42.42.1.




{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: '10.42.42.1'

  config.vm.provision "shell", inline: <<-SHELL
    
    # Apply the latest OS updates
    sudo apt-get update
    sudo apt-get upgrade

    # Install Python goodies
    sudo apt-get install --assume-yes python-pip
    sudo pip  install flask
    sudo pip  install flask-login
    sudo pip  install flask-openid
    sudo pip  install flask-mail
    sudo pip  install flask-sqlalchemy
    sudo pip  install sqlalchemy-migrate
    sudo pip  install flask-whooshalchemy
    sudo pip  install flask-wtf
    sudo pip  install flask-babel
    sudo pip  install guess_language
    sudo pip  install flipflop
    sudo pip  install coverage

  SHELL

end
{% endhighlight %}



