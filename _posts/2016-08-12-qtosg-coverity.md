---
title: "Coverity and Travis CI integration set up for a project with Qt and OpenSceneGraph dependencies"
categories: 
    - Tutorials
excerpt: A continuation post on continuous integration - this time for defect search in your codes. 
tags: 
    - OpenSceneGraph 
    - Qt
    - TravisCI
    - Coverity
    - github
---

{% include toc %}

# Overview

This tutorial is a continuation of the [part 1](http://vicrucann.github.io/tutorials/qtosg-travis/). Here I will show how to set up the compiles files from TravisCI for the analysis of [Coverity Scan](https://scan.coverity.com).

While the [official TravisCI Integration guide](https://scan.coverity.com/travis_ci) provides all the necessary information on how to perform it step by step, I will concentrate only on the parts that needs special attention, or things that caused me some trouble.

To recap, these are the generic steps that needs to be followed for an initial set up: 

* Create a new github branch called `coverity_scan` which will be analyzed by Coverity whenever it is pushed on the gihub.
* Create an account at [https://scan.coverity.com](https://scan.coverity.com) by signing up using your github account.
* Create file `.travis.yml` as was discussed in [part 1](http://vicrucann.github.io/tutorials/qtosg-travis/) of the tutorial.
* Merge the changes to the `coverity_scane` branch from `master` branch.
* Paste the generated Coverity settings into `.travis.yml` files such as project settings, secure key, etc.

After doing the above steps, we are now ready to do final edits of the `.travis.yml` file.

# Changes of `.travis.yml` 

Most of the yml file will remain the same, and we only need to specify what are the build and pre-build commands of the Coverity. 

For the pre-build part, we need to specified a compiler type and the version. It is so that to avoid a warning when no files are emitted for the Coverity analysis. For the build part, we use the `make` command - same way as when we do builds for TravisCI. As a result, this is how `coverity_scan` addon looks like:

{%highlight bash%}
addons:
  apt:
    packages:
      - cmake
      - g++-4.8

    coverity_scan:
      project:
        name: "vicrucann/QtOSG-hello"
        description: "Build submitted via Travis CI of Qt + OpenSceneGraph application"
      notification_email: vicrucann@gmail.com
      build_command_prepend: "cov-configure --comptype gcc --compiler gcc-4.8"
      build_command:   "make VERBOSE=1"
      branch_pattern: coverity_scan
{%endhighlight%}

The further steps of `.travis.yml` remain the same: before the install, installation and before script. For the `script` part, now that we send the files for Coverity scan with its own build command, we do not need to proceed. To avoid re-running the `make` command once again, we check for a git branch name, and if it is `coverity_scan`, we exit:

{%highlight bash%}
script:
  - if [[ "${COVERITY_SCAN_BRANCH}" == 1 ]];
      then
        echo "Don't build on coverty_scan branch.";
        exit 0;
    fi
  - make
{%endhighlight%}

# Project example

I used the same [QtOSG-hello example](https://github.com/vicrucann/QtOSG-hello) as in [part 1](http://vicrucann.github.io/tutorials/qtosg-travis/) of the tutorial. Check the `coverity_scan` branch for the `.travis.yml` file. After I pushed my `coverity_scan` branch to the github, it caused Coverity Scan to perform the analysis. 

Unfortunately, Coverity Scan does not allow scans for test projects, so I could not keep the project at my Coverity Scan account. As a proof of the concept, I only have this screenshot: 

![Coverity passed]({{ site.url }}{{ site.baseurl }}/assets/images/qtosg-coverity/coverity.png)
{: .align-center}

One of my bigger projects rely on Coverity Scan for defect search, so I put a link for that project here too:

[![Coverity Status](https://scan.coverity.com/projects/9322/badge.svg)](https://scan.coverity.com/projects/vicrucann-cherish)


