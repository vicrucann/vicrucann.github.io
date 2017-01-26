---
title: "rshell-mat"
excerpt: "Shell script based imitation of Matlab big data distributor."
header:
  image: /assets/images/teaser-ipag.jpg
  teaser: /assets/images/cryo3d/cached.png
sidebar:
  - title: "Role"
    image: /assets/images/yale-med-logo.png
    image_alt: "logo"
    text: "Postdoctoral Associate"
  - title: "Responsibilities"
    text: "Design and organization of the system framework. Development and addition of new features."
#gallery:
#  - url: /assets/images/cryo3d/cached.png
#    image_path: assets/images/cryo3d/cached.png
#    alt: "3D reconstructed protein viewable from Chimera."
date: 2015
---

`rshell-mat` is a bash script based utility that helps to ease heavy data processing in Matlab. Its main idea is to send the split data to several remote servers and run the most heavy computations simultaneously using those remotes. When the processing is done, the split result files are copied back to the local machine, merged by using the user-provided function; so the data can be used further in Matlab.

`rshell-map` is a working part of [`cryo3d`](https://vicrucann.github.io/minimal-mistakes/portfolio/cryo3d/) framework. The Shell/Matlab code can be found at the [corresponding github repository](https://github.com/vicrucann/rshell-mat).