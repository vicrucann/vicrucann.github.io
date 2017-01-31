---
title: "Tiny library that performs curve fitting based on Schneider's algorithm"
categories: 
    - Tutorials
excerpt: An update blog on my recently implemented library for path fitting that uses C++11 and OpenSceneGraph (for visualization)
tags: 
    - OpenSceneGraph 
    - library
    - C++11
---

{% include toc %}

<figure class="half">
	<img src="/images/curve-fitting/traced.png" alt="image">
	<img src="/images/curve-fitting/fitted.png" alt="image">
	<figcaption>Screenshots of drawn line (left) and curve fitted (right). Note how some details and noise are lost due to the larger fitting threshold.</figcaption>
</figure>

# Overview

For one of my applications, I've been searching for a lightweight C++-based library that would allow me to perform curve fitting. That is, given a set of points, fit those points to a set of adjacent curves. An example of application is when user is drawing on the screen, and the curve fitting helps to smoothen the output. Such algorithms are often used within the drawing and sketching applications. After I failed to find an easy-to-use and modern library, I gave up and ended up writing such "library" myself. 

The algorithm of the library class is based on the Philip J. Schneider paper titled *An algorithm for automatically fitting digitized curves* which was published in *Graphics gems* in 1990. The main idea of the algorithm is given a set of points that belong to a single path, we try to iteratively fit a curve within given threshold. If it is not possible, the curve is split into parts and the process is repeated. For more details, refer to the [original paper](http://dl.acm.org/citation.cfm?id=90941&CFID=671275411&CFTOKEN=85204575). 

As often, the code can be found on my github account - [CurveFitting](https://github.com/vicrucann/CurveFitting).

# Implementation details

The curve fitting code is a template class `PathFitter` which must be sub-classed in order to use the fitting algorithm. In the provided example, I used OpenSceneGraph library for visualization and also used OSG data types such as `Vec3Array` and `Vec3f` for the base class templates. The OSG vectors already provide basic vector functionality such as dot product, normalization and vector length. It is possible to provide any other custom non-OSG based class, but the aforementioned vector functionality must be implemented by the user.

To run an existing example using the OSG data types, we can use the `OsgPathFitter` class which is ready to be used, and provides an example of the subclassing the base class. More details on implementation and sub-classing are provided in the [README](https://github.com/vicrucann/CurveFitting/blob/master/README.md) of the project repository.




