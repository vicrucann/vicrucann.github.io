---
title: "TravisCI set up for a project with Qt and OpenSceneGraph dependencies"
categories: 
    - Tutorials
excerpt: How to integrate test automation into one of your github repositories, where code is based on Qt and OpenSceneGraph libraries. 
tags: 
    - OpenSceneGraph 
    - TravisCI
    - github
---

{% include toc %}

# Overview

This tutorial provides a method on how to use Travis for your code project which is dependant upon Qt and OpenSceneGraph libraries. It is mainly aimed towards beginners who never dealt with Travis files, and as a result the full script example will be provided for a minimalistic code project. Note, there could be always few ways to solve the problem, and it is only one of them; hopefully, simple and intuitive.

Also, this tutorial is not aimed on providing definitions and explaining how Travis works; there is always official documentation for this.

# Travis

While [Travis web-site](https://docs.travis-ci.com/) contains very detailed tutorials on how to do all kinds of setups, as a beginner I found it challenging to set it up for my project which is dependent on Qt and OpenSceneGraph. It is possible to scrape some information and examples on [how to use Travis for Qt projects](https://www.google.com/search?q=travis+qt&ie=utf-8&oe=utf-8#q=travis+for+qt+project), however it took me a while to figure out the best procedure for OpenSceneGraph. 

## Travis setup for Qt dependency

Keeping in mind that Travis uses Ubuntu OS for the builds, we can take advantage of the [available Qt ppa](https://launchpad.net/~beineri), and perform installation from there.

Inside the project's `.travis.yml`, before the installation commands, we add the chosen repository (we chose Trusty package since that's the distribution used by Travis):

{%highlight bash%}
before_install:
    - sudo apt-add-repository --yes ppa:beineri/opt-qt551-trusty
    - sudo apt-get -qq update
{%endhighlight%}

Once the ppa is added as a repository, we can install the necessary Qt packages, for example:

{%highlight bash%}
install:
    - sudo apt-get --yes install qt55base qt55imageformats qt55svg
{%endhighlight%}

After that, we need to make sure CMake knows where the Qt packages are located. We need to edit the `$PATH` variable for that. In `.travis.yml` this can be put under the `before_script` part, e.g.:

{%highlight bash%}
before_script:
    - QTDIR="/opt/qt55"
    - PATH="$QTDIR/bin:$PATH"
    - source /opt/qt55/bin/qt55-env.sh
{%endhighlight%}

And this is all for the Qt part! Depending on the build tool, you might need to run `cmake` or `qmake` commands to prepare the `Makefile` so that the `script` part of the Travis file would only have to run `make`. 

## Travis setup for OpenSceneGraph dependency

Unfortunately, there are no ppa packages for OpenSceneGraph like there are for Qt. One of the best options would be to introduce them, however, there is an easier work-around that requires less time. 

### Preparation of OpenSceneGraph package

The solution I approached was to build OSG for Linux and upload the built files into my github repository. Then for each project commit, Travis would download the installation files, set them up and use to perform the project's compilation and testing (if needed). The steps are described in more details below.

#### OpenSceneGraph build on Linux 

I do not have access to the Linux environment on day to day work basis. I decided to use [cloud9](http://vicrucann.github.io/blog/jekyll-window-alternative/) for my OSG builds, and it worked out great. If you have a Linux machine available, then you can follow official documentation steps for OSG build. 

#### OpenSceneGraph release archive

Assuming you had [built and installed the OSG library on Linux](http://vicrucann.github.io/tutorials/osg-linux-quick-install/), now is the time to prepare an archive containing your version of the OSG. We will copy our "distribution" into a folder called `osg-340-deb`. These are the commands I used:

{%highlight bash%}
cp -vr /usr/local/lib64 osg-340-deb/lib64
cp -vr /usr/local/bin osg-340-deb/bin
cp -vr /usr/local/include osg-340-deb/include
{%endhighlight%}

Now is the time to archive the folder:

{%highlight bash%}
tar -zcvf osg-340-deb.tar.gz
{%endhighlight%}

Upload the package, for example, to your github account. In my case, I forked an official [github OpenSceneGraph repository](https://github.com/openscenegraph/OpenSceneGraph) and [uploaded](https://github.com/vicrucann/OpenSceneGraph/releases/download/OpenSceneGraph-3.4.0/osg-340-deb.tar.gz) (the link will start archive download) the `.tar.gz` file under the tag of the version `3.4.0`. The total archive size came out to be about 11 Mb.

#### Travis commands

Before doing the installation, we need to obtain the necessary OSG release:

{%highlight bash%}
before_install:
    - wget https://github.com/vicrucann/OpenSceneGraph/releases/download/OpenSceneGraph-3.4.0/osg-340-deb.tar.gz
    - tar -zxvf osg-340-deb.tar.gz
{%endhighlight%}

The installation step includes copying the release files into the standard location where CMake will be searching for OSG files:

{%highlight bash%}
install:
    - sudo cp -rv osg-340-deb/lib64 /usr/local/
    - sudo cp -rv osg-340-deb/include /usr/local/
    - sudo cp -rv osg-340-deb/bin /usr/local/
{%endhighlight%}

Before running the script and `cmake` command, we need to make sure the `$PATH` variables are set correctly:

{%highlight bash%}
before_script:
    - export LD_LIBRARY_PATH="/usr/local/lib64:/usr/local/lib:$LD_LIBRARY_PATH"
    - export OPENTHREADS_INC_DIR="/usr/local/include"
    - export OPENTHREADS_LIB_DIR="/usr/local/lib64:/usr/local/lib"
    - export PATH="$OPENTHREADS_LIB_DIR:$PATH"
{%endhighlight%}

And this is all you need for the OpenSceneGraph part. 

# Project example

I had prepared a [small "Hello World" project](https://github.com/vicrucann/QtOSG-hello) which is based on Qt and OpenSceneGraph. The main repository includes [`.travis.yml`](https://github.com/vicrucann/QtOSG-hello/blob/master/.travis.yml) file which summarizes all the code discussed above. As you can see from the project's description, below the file tree, a proof of Travis doing its work is placed: 

[![Build Status](https://travis-ci.org/vicrucann/QtOSG-hello.svg?branch=master)](https://travis-ci.org/vicrucann/QtOSG-hello). 
