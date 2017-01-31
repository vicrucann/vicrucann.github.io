---
title: "How to prepare a Windows binary that uses Qt and OpenSceneGraph"
categories: 
    - Tutorials
excerpt: The tutorial demonstrates how to quickly assemble a stand-alone binary with the list of potentially necessary dll files so that the binary can run on another Windows machine (of the same version).
tags: 
    - OpenSceneGraph 
    - Qt
    - MSVC
---

{% include toc %}

# Problem statement

In this tutorial we will look at how to quickly assemble a binary that can be run on another machine. As an example, assume: your friend would like to give a quick test of your software on their machine. For this purpose, it does not make much sense for your friend to spend time and effort into compiling your code from source (which also means libraries installation and environment setup), but rather to use already compiled executable of your program. One of the biggest assumption is that **both of yours and friend's machines have the same Windows version and platform**.

If you use Qt and OpenSceneGraph for development of your software, you would have to join a bunch of files to accompany your executable. Those files are called libraries and have an extension of `.dll`. If you ever encountered an error which says `dll cannot be found` when you run any program, it means some libraries are missing. 

I am going to provide a **list of some `dll` files** that will most probably be useful for any Qt + OpenSceneGraph project, as well as an example of the **folder structure** where you place your binary. With all that, it is straightforward to copy the necessary files and then run an executable of quite sophisticated program.

# Folder structure

For the majority of `dll` files, the rule is pretty simple: they must be placed same folder where the binary is. 

## OSG plugins

If you use some plugins, for example, your program uses OSG plugins to read third-party files, or Qt plugins to display icons, you have to create a plugin folder with the same name as in the `bin` directory. 

Much easier to see it on an example: assume you are using some serialization plugins of OSG. Your program will not function properly without the necessary plugins, so you have to copy them: 

* first, go to the OSG installation folder
* from the root installation folder, go to `\bin` folder
* there you will notice a plugin folder, e.g. `\osgPlugins-3.4.0`; create folder with the same name at your executable folder
* copy the necessary `dll` files; to be safe I always copy all the `dll` files from plugin folder

## Qt plugins

Same principle follows any Qt plugins that you use. Normally, the Qt's plugins are located at `$QT_ROOT_PATH/plugins/used_plugin_folder`. For example, `imageformats` contain libraries to read different image file formats. If you use any plugin from `imageformats` folder, you need to create a folder `imageformats` in the directory of your executable and then copy the necessary `dll`s. 

## Other library files

This is the list of other `dll` files that may be necessary to run a Qt executable:

* library files of folder `$QT_ROOT_PATH/plugins/platforms` which need to be copied to the corresponding `platforms` folder of executable directory
* libraries such as `icudt53.dll`, `icuin53.dll`, `icuuc53.dll` (the number at the end can differ) - located at the `bin` directory of Qt root folder
* `lib*GL*.dll` and other OpenGL-related files that can be found in `bin` directory of Qt root folder

## A folder structure example

After I assembled a folder for an executable of my CAD-like program, these are some files that ended up in my `exe` folder:

![Structure]({{ site.url }}{{ site.baseurl }}/assets/images/qtosg-binary/structure.png)
{: .align-center}

## Edit

As I found out later, that list is not complete in case if there is no Visual Studio installed. Depending on the MSVC version which was used for compillation, the next files are needed to be added to the root folder:

* `msvcpXYZ.dll` where `XYZ` denotes a number, for example `msvcp140.dll` for Visual Studio 2013
* `VCRuntimeXYZ.dll` with the same `XYZ` notation, e.g., `VCRuntime140.dll`

Both of these files are located at `C:/Windows/SysWOW64/`.

