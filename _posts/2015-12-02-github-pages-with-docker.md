---
categories: Github Pages,Docker, Markdown, Jekyll
date:   2015-12-02 09:33:59 -0600
layout: post
tags: blogging, docker, markdown, jekyll
title:  "Docker Blogging Environment using Github Pages"
---

My time spent at [StackEngine](http://stackengine.com) as an Evangelist was a 
tremendous learning experience. I learned that I am not cut out for the travel
or the constant crowds, but I also realized that I enjoy writing and 
occasionally speaking.

I wanted a way to write useful content and keep track of the things I learn 
in my professional journey and personal hacking.

So, for those of you longing to blog with markdown and leverage Github Pages, 
here are the basics I used to get up and developing in with a Docker run time.

# Prerequisite Knowledge

This is not a Docker tutorial. If you are brand new to Docker I suggest 
reading and playing along with the 
[StackEngine series](http://stackengine.com/docker-101-01-docker-development-environments/)
 I wrote awhile ago.

 If you prefer a video, check out the 
 [great stuff](https://blog.docker.com/2015/03/docker-tutorial-1-installing-docker/) 
 from [John Willis](https://twitter.com/botchagalupe) at Docker.

# Preliminary Tooling

This article is strictly about developing content on a Mac. Given the ease
of using the Docker Toolbox, I suspect it is easy to do the same in Windows. 
If you are using windows, follow these instructions, take some notes and 
write an article to help you tribe.

The basic steps are as follows:

* Install Docker Toolbox
* Launch a Guest

### Mental Model and Benefits



### Install Docker Toolbox

### Launch the Guest Host


# Docker Run time

You now have your jekyll run time in a docker image, but your blog content 
is still on your local machine. Use whatever tool you like.



{% highlight bash %}
docker run \
  --env FORCE_POLLING=true \
  --env JEKYLL_ENV=development \
  --env VERBOSE=true \
  --interactive \
  --label=jekyll \
  --tty \
  --publish $(docker-machine ip jekyll):4000:80 \  
  --rm \  
  --volume=$(pwd):/srv/jekyll jekyll/jekyll:pages \
  jekyll build --watch
{% endhighlight %}

# Conclusion

I am concluded, but still changing 

4000:4000 working now?
