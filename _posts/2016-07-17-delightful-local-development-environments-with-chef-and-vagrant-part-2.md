---
categories: 
- Vagrant
- Chef
- Development Environment
date: 2016-07-17
layout: post
tags: 
- Vagrant
- Chef
- Development Environment
- DevOps
- OSX
title:  "Delightful Local Development Environments with Chef and Vagrant - Part 2"
---

This is the second post in a series of article describing how to set up a local development environment for a simple microservice architecture using Vagrant and Chef.

The goal of the series is to have a fully functional development environment with a basic understanding of the concepts of virtualization and configuration management .  

This series of articles was written to support the development efforts of [Kasasa LTD](https://kasasa.com) in Austin, TX.  Though your tech stack may differ, you should be able to used this series as a way to set up your own teams' development environment in your organization.

# Table of Contents

*  Part 1: [Hello Vagrant!](/vagrant/chef/development%20environment/2016/06/25/delightful-local-development-environments-with-chef-and-vagrant-part-1.html) 
* **Part 2: [Provisioner Basics](/vagrant/chef/development%20environment/2016/07/17/delightful-local-development-environments-with-chef-and-vagrant-part-2.html)**

# Goals for this Article

At the end of this article you should have acomplished the followign tasks: 

* Demonstrate the disposability of this environment
* Use the Vagrant SHELL provisioner to configure the guest machine
* Use Vagrant networking to have a simple Hello World! applicaiton respond in our browser.
* Show that you can develop locally and have your changes show up via the guest runtime environment.
* Show that you can onboard to a new project in minutes.

# First Princples

For this article it will be important to keep these definitions straight:

* A "provisioner" is a Vagrant construct that allows access to the guest for a set of headless commands to run. This is how a guest gets automatically configured
* A "provider" is the software (Virtualbox) used to _provide_ the hypervisor for Vagrant to launch machine in.

In reality the above is quite a jump in implementation. Nobody uses a base Ubuntu install. We have to install our runtime, monitoring agents, logging paradigm, etc.  

# Hello Again Vagrant

Before we get to configuring our guest, let's justify the work we just did and are about to do.

In our last article we created the directory `~/kumbaya` and initialized Vagrant there.  Move back to this directory and make sure you are in the same state as this article.

{% highlight bash %}
behemphi:~/kumbaya 21:38:03 > vagrant status
[SNIP]
behemphi:~/kumbaya 21:38:16 > vagrant destroy
==> default: VM not created. Moving on...
behemphi:~/kumbaya 21:38:22 > vagrant up
{% endhighlight %}

**What Happened?**

Since we were in an unknown state, it was a good idea to check on the status of the guest.  Your results will vary, but they will be descriptive. Take the time to read the output.  

Regardless of the state of the guest we took the decsion to destroy it.  In the case above there was no guest running to Vagrant tells me this is the case. 

Finally we start a new guest. 

You have just completely trashed and recreated your runtime environment.  Consider how long it would take to do this on the bespoke set up most developers have on their workstations (rather than guest).

**--**

If you were to share this repo with me, I could easily clone it, start the guest and add value.  Consider this on a team of five developers. An update to the runtime environment is a simple pull.  The guest can be destroyed at any time without losing a single line of code!

Wait, that is _not_ true.  Right now we would have to log in and configure the machine at the CLI.  This would be manual, error prone and generally not useful.

Let's make it true.

## Sharing Configuration

Let's set up this guest to run a Python applicaiton.  The [Flask Mega Tutorial](http://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world) is some of the best writing out there, so we will leverage it. 

First lets's update our Vagrantfile to do a bit more.

{% highlight ruby %}
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"

  config.vm.network :private_network, ip: '10.42.42.10'

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

Paste the above into your `~/kumbaya/Vagrantfile` and issue a `vagrant provision` command. This time things take much longer and there 100's of lines of green output. 

**What Happened?**

When Vagrant launches the guest, it assigns it a private IP adderss of 10.42.42.10.  This will help with local development. Scroll up into the white output and find where the IP assignment happens.

After Vagrant launched the guest, it executed the shell commands found in the provisioner section of the file. 

**--**

**Gotcha!**

This is a powerful demonstration, but its a very blunt instrument. You should not attempt to re-run the SHELL provisioner because there is no guarentee that any shell command will have the expected effect a second time through. Better to destroy the machine and `vagrant up` again for now.

Destroying and "upping" gets to be time consuming. In a future article, Chef will replace the generic shell provider to address this very problem.

**--**

It should now be clear that you are able to install a run time - Python's Flask in this case - and work with anyone on your team to contribute to the run time.  This exact shell script can be used to configure the production application servers.  Envrionment parity for the win!

## Hello Flask

Let's create a simple "Hello World" application with Flask. First we need to lay down three python files:

`~/kumbaya/run.py`
{% highlight python %}
#!/usr/bin/env python

from app import app
if __name__ == '__main__':
    app.run()
{% endhighlight %}

`~/kumbaya/app/__init__.py`
{% highlight python %}
from flask import Flask

app = Flask(__name__)
from app import views
{% endhighlight %}

`~/kumbaya/app/views.py`
{% highlight python %}
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
{% endhighlight %}

If you were running the Flask application on your workstation, you would have to visit the address `127.0.0.1:5000`, but we have configure Vagrant to explicitly use the IP address 10.42.42.10.

In order to see the results you should start up the app from the guest:

```
behemphi:~/kumbaya 21:42:53 > vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ export FLASK_APP=/vagrant/run.py
vagrant@vagrant-ubuntu-trusty-64:~$ export FLASK_DEBUG=1
vagrant@vagrant-ubuntu-trusty-64:~$ python -m flask run --host=0.0.0.0
 * Serving Flask app "run"
 * Forcing debug mode on
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 311-149-961
```

**What Happened?**

From inside the guest you started the application. 

Verify it is running by browsing to `[http://10.42.42.10:5000/](http://10.42.42.10:5000)`

**--**

## Develop Local Run Virtual

With the flask application running, open the file `~/kumbaya/app/views.py` and change the "Hello World!" text to "Hello Vagrant, would you like a Flask?" and save.

You will see output like the following:

```
 * Detected change in '/vagrant/app/views.py', reloading
 * Restarting with stat
 * Debugger is active!
 * Debugger pin code: 311-149-961
```
Refresh the browser and  your new message will appear.

**What Happened?**

The Flask process saw the files change on the _guest_ even though you changed them on the workstation. This is because Vagrant mounted the workstation location `~/kumbaya/` to the guest location `/vagrant`. 

**--**

## New Developer Workflow

Let's move you personal version of `kumbaya` out of the way with `mv ~/kumbaya ~/kumbyebye`.  

Let's assume for a moment you are a new hire on the kumbaya project.  As a team we have agreed to keep our projects in our home directory. To get started developing on the Project Kumbaya:

{% highlight bash %}
behemphi:~ 22:43:07 > cd ~
behemphi:~ 22:43:09 > git clone git@github.com:behemphi/kumbaya.git
Cloning into 'kumbaya'...
[snip]
{% endhighlight %}

As an exercise, move in to the `kumbaya` directory, start the guest and change the, "Hello world!" message as we did above.  

**What Happened!**

With Vagrant set up, you just onboarded yourself to a project in less than 5 minutes.  

**--**

# Wrapping Up 

You have thrown the environment away then restored it in moments.  Using the SHELL provisioner, you updated the OS and added a python runtime environment. To ensure it is easy to find the machine, you assigned it a private IP on your local machine.

With a production-like machine running, you ran the code mounted from your workstation via the guest.  You manipulated the text of a message and saw the result in the browser.  

Finally, to prove it all over again, you threw away not just the runtime environment, but the application as well.  You cloned my repo and performed the same set of actions, thus proving you are truly able to onboard yourself to a new project in minutes.

# Looking Forward

In the next article we will install the Chef Development Kit (ChefDK) and wire it in to Vagrant. This will allow us to become more sophisticated in how we configure our runtime environment.  


