---
title: "OpenSceneGraph intersectors example: line, point and virtual plane"
categories: 
    - Tutorials
excerpt: Some previous tutorials combined into a single example; applicable in CAD-like environment.
tags: 
    - OpenSceneGraph 
    - 3D geometry
---

{% include toc %}

# Goal

![Intersectors figure]({{ site.url }}{{ site.baseurl }}/assets/images/osg-intersectors/intersectors.gif)
{: .align-center}

When I was building CAD-like program, one of the essential needs was different types of intersectors. The purpose was to be able to select different elements such as points, lines and drag them in a special manner, e.g., so that dragged rectangle point remains in the same plane as the rectangle. 

The [tiny demo program](https://github.com/vicrucann/osg-intersectors-example) I created demonstrates how to use exactly the aforementioned intersectors. In short, I provide three types of intersectors:

1. Line intersector - i.e., the user is able to select lines.
2. Point intersector - i.e., the user is able to select points.
3. Virtual plane intersector - i.e., the ray cast always intersects with a virtual plane to define a virtual intersection point.

This post's main goal is to describe the created demo. All the parts of the demo are based on previous tutorials which will provide in depth details, if necessary.

# Intersectors

## Line intersector

Line intersection is based on finding the shortest distance between the raycast and all the line elements on the scene. For more details on how the line intersector works, feel free to check [part 1](http://vicrucann.github.io/tutorials/osg-stroke-intersector/) and [part 2](http://vicrucann.github.io/tutorials/osg-stroke-intersector-improved/) of the corresponding tutorials. 

Within the demo code, the line intersection is in action whenever you observe the wire's color to turn **magenta** color. 

## Point intersector

The point intersection is based on finding the shortest distance between the raycast and all the point elements, and then finding of the distance is within a threshold. This part is heavily based on [Chapter 10 of OpenSceneGraph Cookbook](http://documents.tips/documents/openscenegraph-cookbook-chapter-10.html). In the article you can find all the necessary details on how the method works.

Within the demo code, the point intersection is in action whenever you observe the point's color to turn **bright green**.  That indicates the selected point is within the threshold distance from the ray casted from the camera view through the mouse position.

## Virtual plane intersector

The virtual plane intersector is based on finding an intersection point between the ray that was cast by the mouse with a virtual plane in 3D. An example of application of such intersector could be [drawing on plane in 3D](http://vicrucann.github.io/tutorials/osg-raycast/). For the demo, we use it to assure the dragged point elements stay within the same plane as the rectangle. More precisely, the intersector helps to find a new position of the wire corner by finding an intersection between the mouse ray and the virtual plane of the rectangle. 

Within the demo, the intersector is used whenever the mouse drags a point. In this case, it is indicated by **yellow** color. 
