---
title: "GLSL shader that draws a Bezier line given four control points"
categories: 
    - Tutorials
excerpt: By using OpenSceneGraph and GLSL shader, this code snippet demonstrates how to draw a Bezier line from the given control points. It works as well in 3D space and is well suited for applications such as curve drawing and fitting.
tags: 
    - OpenSceneGraph 
    - OpenGL
    - GLSL
    - 3D geometry
mathjax: true
---

{% include toc %}

# Context

This post is a continuation of one of the previous examples on [how to draw thick and smooth lines in 3D space](http://vicrucann.github.io/tutorials/osg-shader-3dlines/). Now we want to be able to not just draw a straight line, but a curve. As an example, the curve can be represented by a set of Bezier curves which were obtained by using a [curve fitting](http://vicrucann.github.io/blog/curve-fitting-c++/) algorithm. So the main purpose of this post is to provide an example *code snippet* of a **GLSL shader** that is being able to:

1. Draw thick and smooth lines in 3D by [turning `GL_LINE_STRIP_ADJACENCY` into triangular strip](http://vicrucann.github.io/tutorials/osg-shader-3dlines/)
2. Sample the curve data from the given control points

In this post it will be shown how we can use the code from the point 1 and extend it to drawing curves in 3D instead of lines and polylines.

# Drawing a Bezier curve

The [Bezier curve](https://en.wikipedia.org/wiki/B%C3%A9zier_curve) is represented by two endpoints and two control points. Therefore, in order to pass to the shader a Bezier curve (or a set of the curves), we have to provide all the control points. 

The modification of the given shader is straightforward given the cubic Bezier curve formula:

\\[B(t)=(1-t)^3P_0+3(1-t)^2tP_1+3(1-t)t^2P_2+t^3P_3\\]

Where \\(P_i\\) is one of the four control points for a given Bezier curve.

We incorporate the given formula into the functions to use inside our GLSL code:

{%highlight cpp%}
vec4 toBezier(float delta, int i, vec4 P0, vec4 P1, vec4 P2, vec4 P3)
{
    float t = delta * float(i);
    float t2 = t * t;
    float one_minus_t = 1.0 - t;
    float one_minus_t2 = one_minus_t * one_minus_t;
    return (P0 * one_minus_t2 * one_minus_t + P1 * 3.0 * t * one_minus_t2 + P2 * 3.0 * t2 * one_minus_t + P3 * t2 * t);
}
{%endhighlight%}

Most of the code stays intact, with the exception of the loop part when we go through all the passed vertices. Now we haven to take into account how many segments there should be in the curve. The introduced variable `nSegments` can be passed to the shader by means of uniforms or set to constant values inside the shader. The pseudo-code of the loop of the main function will look like following:

{%highlight cpp%}
for (int i=0; i<=nSegments; ++i){
    // Sample the control points to the curve points
    Points[i] = toBezier(delta, i, ...);

    // interpolate the colors
    colors[i] = ... ;

    // transform to the screen coordinate space
    points[i] = toScreenSpace(Points[i]);

    // extract z-values so that the drawing order remains correct
    zValues[i] = toZValue(Points[i]); 

    // finally send all the info for drawing procedure
    drawSegment(points, color, zValues);
}
{%endhighlight%}

A few words on the color interpolation. For that part the main idea is to define to which  of the three Bezier fragment the current point belongs to, and then interpolate between the two colors of endpoints of that Bezier segment based on the location of the point to the endpoints of the segment. Refer to the source code for concrete example. 

# Codes

[shader-3dcurve](https://github.com/vicrucann/shader-3dcurve) github repository contains the minimalistic OpenSceneGraph scene and the GLSL shaders that help to generate two curves located in different 3D planes: 

<figure class="half">
    <img src="/assets/images/osg-shader-3dlines/curve-1.png" alt="image">
    <img src="/assets/images/osg-shader-3dlines/curve-2.png" alt="image">
</figure>