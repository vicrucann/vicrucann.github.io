---
title: "Quick steps for OpenSceneGraph installation on Linux"
categories: 
  - Tutorials
excerpt: Detailed steps to download, compile and install the library; management of environment variables and example compilation.
tags: 
  - OpenSceneGraph
  - Linux
---

# Minimal steps for Linux

For my work I had to install OSG on both Linux and Windows. It took me a while to figure things out for Windows platform. As a contrary, the installation on Linux was rather straightforward and went smooth. Here I will describe all the steps it took for me to install the OSG on Linux.

The [official documentation](http://www.openscenegraph.org/index.php/documentation/getting-started) and [reference books](http://www.openscenegraph.org/index.php/documentation/books) provide the full steps on how to install the OSG library on different platforms. Although binary installation files available for older OSG version, I strongly recommend installation from the source - that way you will get the latest version, as well as will be able to refer to the source directly when needed. 

I intend to provide a very brief guide for Linux that worked for my laptop (Lubuntu-14.04). It is supposed to be concise, however if you run into a difficulty, better to check with the official guide.

## Download

To download the latest stable version (or the latest development version if you wish), clone from the [official OpenSceneGraph github repository](https://github.com/openscenegraph/osg):

{% highlight bash %}
~ $ git clone https://github.com/openscenegraph/osg
{% endhighlight %}

Assuming you want to use the latest stable verion of OSG, switch the branch inside the local repository:

{% highlight bash %}
~ $ cd osg
~/osg $ git checkout tags/OpenSceneGraph-x.y.z
{% endhighlight %}

Replace the `x.y.z` with the  numbers that correspond to the version you intend to use, e.g., `3.4.0`.

## Install

Now it is the time to run the installation steps which are also specified in a corresponding `README` file:

{% highlight bash %}
~/osg $ mkdir build
~/osg $ cd build
~/osg/build $ cmake ..
~/osg/build $ make
~/osg/build $ sudo make install
{% endhighlight %}

The above steps might take a while (from 20 minutes to several hours), depending on how powerful you PC is. In case if you are planning to do the compilation together with the OSG examples, refer below to "Example compilation" section.

### `Bash` variables

After the installation, it is necessary to set certain environment variables that are used by the OSG. To do it, use a text editor of your choice and open `.bashrc` or `.zshrc` or similar file which contains your shell environment information. Now add the following lines:

{% highlight bash %}
export LD_LIBRARY_PATH="/usr/local/lib64:/usr/local/lib:$LD_LIBRARY_PATH"
export OPENTHREADS_INC_DIR="/usr/local/include"
export OPENTHREADS_LIB_DIR="/usr/local/lib64:/usr/local/lib"
export PATH="$OPENTHREADS_LIB_DIR:$PATH"
{% endhighlight %}

Make sure the `lib` and `include` file paths correspond to the path where your OSG libraries were installed (look through the `make install` command output to see the destination of the libraries). In my case the `include` files were installed to `/usr/local/include`, and the `lib` files were installed to `/usr/local/lib64`. 

Before the usage of OSG, restart your bash environment.

### Example compilation

The OSG supplies a bunch of useful examples. You may want to look through them, or compile against them. To install them, run the aforementioned `cmake` command with the next argument:

{% highlight bash %}
~/osg/build $ cmake .. -DBUILD_OSG_EXAMPLES=1
{% endhighlight %}

Then follow the other steps as they are: `make` and `sudo make install` (or you can try to run the examples from the `build` directory directly).

To run the examples, download the data of models and textures that is supplied with OSG. It can be found at their [official github repository](https://github.com/openscenegraph/osg-data). To clone the data using `git`, run the following command:

{% highlight bash %}
~/osg/build $ cd $HOME
~ $ git clone https://github.com/openscenegraph/osg-data
{% endhighlight %}

Then copy the data to a folder designated for OpenSceneGraph data. For this, I created a folder at `/usr/local/OpenSceneGraph/data`, and then copied the `osg-data` folder content to the designated folder. As a last step, include the data path as an environment variable (edit the `.bashrc` file again):

{% highlight bash%}
export OSG_FILE_PATH="/usr/local/OpenSceneGraph/data:/usr/local/OpenSceneGraph/data/Images"
{% endhighlight %}
 
Do not forget to restart your bash environment. Now your OSG library is set up and ready to be used.

