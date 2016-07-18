---
categories: 
- github-pages
- docker
date:   2015-12-02 09:33:59 -0600
layout: post
tags: 
- blogging
- docker
- markdown
- jekyll
title:  "Docker Blogging Environment using Github Pages"
---


My time spent at [StackEngine](http://stackengine.com) as an Evangelist was a  tremendous learning experience. I am not cut out for the travel or the constant crowds. But I do enjoy writing and, occasionally, speaking.

Wordpress hurts my soul. No source control, no work flow, markdown support is dogdgy at best. I want to be able to blog in the same way I code. Github Pages, Jekyll and Docker for the win.

If you want to skip the background and get right to it, jump down to [Preliminary Tooling]({% post_url 2015-12-02-github-pages-with-docker %}#preliminary-tooling) to get started.

## Philosophy

I want to blog in a fashion that appears frictionless to me. 


### Goals and Research

Setting up a, free, hosted static site at Github is a snap with [Github Pages (GHP)](https://pages.github.com). Check out the link to get started.

The first thing I noticed was that the basic workflow is to commit and push to production. The little DevOps angel sitting on my right shoulder cringed and started gibbering about culture, while the the little GSD (get stuff done) devil on my left shoulder danced and sang, "Just do it ... swoosh!"

I landed on the side of the angels for two reasons:

* GHP is not just a blog organizer, it is an entire site with themes and content pages. I needed a way to try things outside of a production environment.
* I want to stage changes so I can edit them before they are live. You know ... actually care about quality.

I have hopes of building a pipeline with spelling and grammar checking and other nifty things. Ultimately I could use this to explain a CI/CD pipeline to a non-technical person in terms of something they can completely understand.

### Mental Model

Github Pages is based on the [Jekyll](https://github.com/jekyll) static site generator. It's a Ruby gem with a number of dependencies. At this point, even the devil on my left was showing signs of fear. I _hate_ jacking with Ruby directly on my workstation. [There be monsters.](https://en.wikipedia.org/wiki/Here_be_dragons)

Additionally, I love the elegant simplicity of ByWord for writing markdown. I don't want to have to long in to a virtual machine to blog. Setting up such tools on a VM or a container is not value added work.

This means I want the Ruby runtime, gems, etc contained in a virtual environment. I then want to mount the blog site directory from my workstation into the container. I chose to do this with Docker and the folks from Jekyll already have a container for this purpose.

When I am satisfied with the content, I want to commit and push to my `behemphi.github.io` project and have the changes be in production.  

I am using the exact environment described below as I write this text for this minimum viable blogging environment.

#### Shout Out

The guys at Jekyll were beyond helpful as I was getting this stood up and asking a number of questions about Jekyll itself and how their container works. 

## Prerequisite Knowledge

This is not a Docker tutorial. If you are brand new to Docker I suggest reading and playing along with the [StackEngine series](http://stackengine.com/docker-101-01-docker-development-environments/).

If you prefer a video, check out the [great stuff](https://blog.docker.com/2015/03/docker-tutorial-1-installing-docker/) from [John Willis](https://twitter.com/botchagalupe) at Docker.

## Preliminary Tooling

This article is strictly about developing content on a Mac. Given the ease of using the Docker Toolbox, I suspect it is easy to do the same in Windows. If you are using windows, follow these instructions, take some notes and write an article to help your tribe.

### Install Docker Toolbox

[Download the Docker Toolbox](https://www.docker.com/docker-toolbox) and install it on your workstation.

The toolbox will provide you with:

* Virtualbox - A hypervisor for running guest machines on your workstation
* Guest Image - A Docker host with the latest stable version of Docker running
* Docker Machine - A powerful tool for running and managing one or more guest machines
* Docker Compose - A handy tool that allows us to store our run time configuration in the porject for ease of use.

### Launch the Guest Host

To launch a machine named "jekyll-runtime" do the following:

{% highlight bash %}
docker-machine create jekyll-runtime --driver virtualbox
{% endhighlight %}

**Gotcha:** 

If you don't name the machine "jekyll-runtime" be sure to pay careful attention to the commands below and replace the name accordingly.

--

It can take a couple of minutes for the machine to come up so be patient. When the command completes, you _must_ set some environment variables. 

```
behemphi:~/behemphi/behemphi.github.io 11:50:05 > docker-machine env jekyll-runtime
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.101:2376"
export DOCKER_CERT_PATH="/Users/behemphi/.docker/machine/machines/jekyll-runtime"
export DOCKER_MACHINE_NAME="jekyll-runtime"
## Run this command to configure your shell: 
## eval "$(docker-machine env jekyll-runtime)"
```

While cutting and pasting these will work, the easiest way to set the environment is to simply use this command:

{% highlight bash %}
eval $(docker-machine env jekyll-runtime)
{% endhighlight %}

### Trust but Verify

Before we get into a big long Docker command, a sanity check:

```
behemphi:~/behemphi/behemphi.github.io 11:51:35 > docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

This shows that we are getting info from the docker daemon.

**Gotcha:**

The environment is only good for the current terminal window. If you open a second terminal and try to interact with Docker you will get an error message. Simple set the environment variables as done above and you can then connect to the same guest and Docker daemon from a the second terminal.

--

## Github Pages in a Docker Container

Now that Docker is working, we need to stand up our new blog site:

* Run a Docker command to scaffold the new site
* Run a docker command to build and serve blog content
* See the site in the browser
* Write a quick blog and see it in action
* Use Docker Compose to keep our runtime configuration in a sane way


### Jekyll Scaffolding

This step need only be done if you are setting up Jekyll for the very first time:

You will need to create a directory on your workstation for your site named `GITHUB_USERNAME.github.io`. [Here are directions](https://help.github.com/articles/create-a-repo/) if this is not familiar to you.

For this article, we will assume you are in the directory you created. Things will not work for strange reasons if you are not in this directory.

{% highlight bash %}
docker run \
  --interactive \
  --label=jekyll \
  --publish 4000:4000 \
  --rm \
  --tty \
  --volume=$(pwd):/srv/jekyll \
  jekyll/jekyll:pages jekyll new . --force
{% endhighlight %}

This provides a one time command and will create the directories your jekyll 
project. 

**Note:**

The only thing installed on your workstation are directories and files. No Ruby environment or gems were installed locally. However, if you are new to Docker, consider the potential for mischief. _Know what you are doing when you run a Docker container!_

### Running the Docker Container 

Let's start a container and get the rest of the setup done:

{% highlight bash %}
docker run \
  --env FORCE_POLLING=true \
  --env JEKYLL_ENV=development \
  --env VERBOSE=true \
  --interactive \
  --label=jekyll \
  --name=jekyll_runtime \
  --tty \
  --publish "$(docker-machine ip jekyll-runtime):4000:80" \
  --rm \
  --volume="$(pwd):/srv/jekyll" \
  jekyll/jekyll:pages jekyll build --watch
{% endhighlight %}

It will take some time to pull the 150 MB of Ruby goodies, so let's deconstruct the command while you are waiting.

* `--env FORCE_POLLING=true` ... Forces the polling of your workstation directories so you see the results of a change in real time.
* `--env JEKYLL_ENV=development` ... Tells Jekyll to watch the `_drafts` directory 
* `--env VERBOSE=true` ... Causes a line to be written in your terminal each time you save as a visual cue that something happened and that all is well.
* `--interactive` ... docker daemon setting that keeps the terminal in the foreground. Works in conjunction with the VERBOSE environment variable
* `--label=jekyll` ... metadata for the container
* `--name=jekyll_runtime` ... name the container for ease of reference
* `--tty` ... Allows for a smoother terminal connection
* `--publish` "$(docker-machine ip jekyll-runtime):4000:80" ... Gets the IP address of your current guest machine and maps its port 4000 to port 80 in the container where nginx serves the pages
* `--rm` ... Remove the container completely when it exits
* `--volume="$(pwd):/srv/jekyll"` ... Mount your current directory to the contianers directory `/srv/jekyll` which is the webroot for nginx running in the container. 
* `jekyll/jekyll:pages jekyll build --watch` ... Using the jekyll/jekyll Docker iamge from the public Docker hub, run jekyll in build mode with it watching your directory for changes. Works in conjunction with the FORCE_POLLING environment variable.

Remember, nothing on your host will change here except the content of the `_site` directory where Jekyll places the build output on each save.

### Verify the Environment

To see you are now up, first get the IP address of your guest host:

```
> docker-machine ls
NAME             ACTIVE   DRIVER       STATE     URL                         SWARM   ERRORS
jekyll-runtime   *        virtualbox   Running   tcp://192.168.99.100:2376 
```

From above, you can see it is `192.168.99.100`. In your browser go to [`http://192.168.99.100:4000`](http://192.168.99.100:4000) and you will see the base site. 

### Writing a blog post

In the directory `_posts`, create a new file called `2000-01-01-lorum-ipsum.md` and copy/paste the content below

```
---
layout: post
title: Titlus Ipsumadori
---

## Lorum Heading
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce efficitur sem nec pulvinar facilisis. Morbi pretium risus non sagittis luctus. Praesent eu dictum lacus, sit amet eleifend mi. Suspendisse rutrum est id ex convallis, sit amet dignissim magna consequat. Nulla non nisi eu erat tempus pulvinar nec ac leo. Vestibulum bibendum tellus eu lacus varius iaculis. 

* Proin ac imperdiet metus, eu condimentum nunc. Integer ultrices odio a porttitor maximus. Morbi ac elementum lacus. Pellentesque volutpat nisi sed mauris ultrices, vitae sollicitudin massa vestibulum. Nullam ex est, molestie et turpis vel, congue hendrerit eros. Donec egestas vestibulum viverra. In risus nisi, fermentum quis imperdiet sit amet, dignissim sed quam. In sollicitudin malesuada est ut gravida. In ultrices mi mauris, ut consequat ipsum ultricies sit amet.
* Class aptent taciti sociosqu ad litora torquent per conubia nostra, per inceptos himenaeos. Phasellus varius erat quis pulvinar tempor. Nunc facilisis magna augue, et auctor nibh luctus nec. Curabitur tempor molestie nisi, ac feugiat turpis. Ut pharetra rhoncus nunc. Praesent bibendum felis at condimentum tincidunt. Curabitur vulputate pellentesque metus, sit amet facilisis felis ornare et. Suspendisse magna nisl, commodo vitae pellentesque sed, finibus sed nisi. 

### Ipsum Subheading

Cras vel sem dapibus, commodo lacus ut, lacinia diam. Nulla mollis, nunc ac sagittis dignissim, ipsum orci aliquet erat, eget interdum lorem metus in diam. In vel arcu sapien. Quisque ut justo nec tellus consequat viverra. Cras tempor fermentum sem, in molestie nulla varius id. Quisque sapien erat, tempus sit amet condimentum in, mollis auctor velit. Suspendisse sed ultrices metus, tincidunt convallis ante.
```

When you save the file, you will see a line similar to the one below in the terminal window where the container is running:

```
Regenerating: 1 file(s) changed at 2015-12-13 19:19:12 ...done in 0.131089747 seconds.
```

If you refresh the page, you will see your new post appear. Click on the link and you will see the post itself.  

### Using Docker Compose

Remembering the above `docker run` command each time you would like to work is not very convenient, so let's use another tool from the Docker Toolbox: Docker Compose.  

Compose is meant to be a way to store and execute the runtime command(s) of one or more containers. You can find the most current compose file in the root of this project. It is called `docker-compose.yml`

Here is a copy as of this writting

{% highlight yaml %}
jekyll_runtime:
  image: "jekyll/jekyll:pages"
  environment:
    - FORCE_POLLING=true
    - JEKYLL_ENV=development
    - VERBOSE=true
  container_name: jekyll_runtime
  tty: true
  ports:
    - "$JEKYLL_RUNTIME_IP:4000:80/tcp"
  volumes:
    - ".:/srv/jekyll"
  command: jekyll build --watch
{% endhighlight %}

To use compose, you wlll need to run it as follows:

```
JEKYLL_RUNTIME_IP=$(docker-machine ip jekyll-runtime) docker-compose up
```

Docker Compose does not yet support [variable interpolation](https://github.com/docker/compose/issues/1377), so setting the machine IP first is a reasonable wrokaround.

**Gotcha:**

As of this writing, the container starts just fine, runs for 2 minutes then dies gracefully. It is a consistent behavior I have not yet had time to track down.

--

## Beyond an MVP

While I am happy to be able to blog in a way that is comfortable to me, there are quite a few things I would like to get done:

* Figure out what is going on with Docker Compose
* Install a theme for a bit more personality
* Understand more about how post metadata works
* Understand how SEO works for Github Pages
* Set up a spell checker as part of a pipeline
* Get a vanity URL. 

For now, however, this minimum viable blog will serve its purpose and provide and outlet to write in.