---
title: "cacharr"
excerpt: "Matlab class that allows to work with very large matrices through caching; thus avoid matrix allocation out-of-memory error."
header:
#  image: /assets/images/teaser-ipag.jpg
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
date: 2015-08-01
---

`CachedNDArray` - data structure that allows to deal with large N-dimensional arrays through caching method:
* Allows to avoid Matlab *out-of-memory error* by caching large array into several files on hard disk and then reading the necessary chunks using `memmapfile` function.
* The data structure is inherited from handle abstract class which avoids parameter-by-value and supports parameter-by-reference.
* Supports two types of movements - continuous (very slow) and discrete (fast); the former might have no more than two file to represent a chunk; while the latter means the data is processed chunk-after-chunk, and each chunk is represented strictly as a single file.
* Caching flag can be set to either manual or automatic mode. If no caching is needed to perform, the `CachedNDArray` is treated like a normal Matlab array.
* Automatic or manual dimension breakage into number of chunks.

[View on github](https://github.com/vicrucann/cacharr){: .btn .btn--success}
