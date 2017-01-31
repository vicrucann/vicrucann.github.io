---
title: "GLSL shader for fog imitation"
categories: 
    - Tutorials
excerpt: By using OpenSceneGraph and GLSL shader, this code snippet demonstrates a simplest way to add a fog effect to your geometries by diluting geometry's color into the background color. 
tags: 
    - OpenSceneGraph 
    - OpenGL 
    - GLSL
mathjax: true
---

# Context

The provided code snippet demonstrates the basic idea behind the fog imitation. The chosen fog model is a linear function which means the fog factor increases linearly with the distance from the current camera view. The fog effect is added through a fragment shader. The GLSL shaders are provided together with the OpenSceneGraph basic code that performs a simple polygon drawing. The shaders can be used with any other OpenGL-based code provided the necessary uniforms are passed to the shaders.

# Simple scene 

For the scene, we will add two polygons located in two different 3D planes. We will make them adjacent one to another so that to test occlusion colors:

{%highlight cpp%}
vertices->push_back(osg::Vec3f(0,0,0));
vertices->push_back(osg::Vec3f(0,0,1));
vertices->push_back(osg::Vec3f(1,0,1));
vertices->push_back(osg::Vec3f(1,0,0));
vertices->push_back(osg::Vec3f(1,1,0));
vertices->push_back(osg::Vec3f(0.5,5,0));
{%endhighlight%}

We also provide different colors for each of the vertices and make sure the color biding is set to `BIND_PER_VERTEX`:

{%highlight cpp%}
colors->push_back(osg::Vec4f(0.1, 0.9, 0.1, 1));
colors->push_back(osg::Vec4f(0.2, 0.1, 0.9, 1));
colors->push_back(osg::Vec4f(0.7, 0.9, 0.1, 1));
colors->push_back(osg::Vec4f(0.9, 0.2, 0.9, 1));
colors->push_back(osg::Vec4f(0.9, 0.2, 0.9, 1));
colors->push_back(osg::Vec4f(0.9, 0.9, 0.1, 1));

// ...
geom->setColorArray(colors, osg::Array::BIND_PER_VERTEX);
{%endhighlight%}

# Necessary uniforms 

Just like with [one of the previous shaders](http://vicrucann.github.io/tutorials/osg-shader-3dlines/) for drawing lines in 3D, we will need `ModelViewProjectionMatrix` in order to obtain `gl_Position`. Using OpenSceneGraph, we frame the matrix as a callback which is updated whenever current camera position changes: 

{%highlight cpp%}
struct ModelViewProjectionMatrixCallback: public osg::Uniform::Callback
{
    ModelViewProjectionMatrixCallback(osg::Camera* camera) :
            _camera(camera) {
    }

    virtual void operator()(osg::Uniform* uniform, osg::NodeVisitor* nv) {
        osg::Matrixd viewMatrix = _camera->getViewMatrix();
        osg::Matrixd modelMatrix = osg::computeLocalToWorld(nv->getNodePath());
        osg::Matrixd modelViewProjectionMatrix = modelMatrix * viewMatrix * _camera->getProjectionMatrix();
        uniform->set(modelViewProjectionMatrix);
    }

    osg::Camera* _camera;
};
{%endhighlight%}

Another variable that we will need is the camera's position in 3D space at any time. Once again, we use OpenSceneGraph's callback system to extract camera's eye and pass it as a uniform:

{%highlight cpp%}
struct CameraEyeCallback: public osg::Uniform::Callback
{
    CameraEyeCallback(osg::Camera* camera) :
            _camera(camera) {
    }

    virtual void operator()(osg::Uniform* uniform, osg::NodeVisitor* /*nv*/) {
        osg::Vec3f eye, center, up;
        _camera->getViewMatrixAsLookAt(eye, center, up);
        osg::Vec4f eye_vec = osg::Vec4f(eye.x(), eye.y(), eye.z(), 1);
        uniform->set(eye_vec);
    }
    osg::Camera* _camera;
};
{%endhighlight%}

The both uniforms are added to a state set (`osg::StateSet`) variable of a root scene:

{%highlight cpp%}
osg::Uniform* modelViewProjectionMatrix = new osg::Uniform(osg::Uniform::FLOAT_MAT4, "ModelViewProjectionMatrix");
    modelViewProjectionMatrix->setUpdateCallback(new ModelViewProjectionMatrixCallback(camera));
    state->addUniform(modelViewProjectionMatrix);

    osg::Uniform* cameraEye = new osg::Uniform(osg::Uniform::FLOAT_VEC4, "CameraEye");
    cameraEye->setUpdateCallback(new CameraEyeCallback(camera));
    state->addUniform(cameraEye);
{%endhighlight%}

# GLSL Shaders

For our purpose, we will only need to use vertex and fragment shaders. The vertex shader sets up the correct `gl_Position`, and the fragment shader is where we will introduce color changes in order to imitate foggy environment.

## Vertex shader

The vertex shader is the standard for GLSL version 3.3:

{%highlight cpp%}
#version 330

uniform mat4 ModelViewProjectionMatrix;

layout(location = 0) in vec4 Vertex;
layout(location = 1) in vec4 Color;

out VertexData{
    vec4 mColor;
    vec4 mVertex;
} VertexOut;

void main(void)
{
    VertexOut.mColor = Color;
    VertexOut.mVertex = Vertex;
    gl_Position = ModelViewProjectionMatrix * Vertex;
}
{%endhighlight%}

As an output, we make sure to pass color and 3D coordinates of each vertex. The latter will be used when we will be calculating the distance between the camera's eye and the vertex so that to assign fog color.

## Fragment shader

We derive the fog color as a mix between the passed color of geometry and background color. In the fragment shader, we also have to pass `FogColor` as a uniform:

{%highlight cpp%}
state->addUniform(new osg::Uniform("FogColor", FOG_COLOR));
{%endhighlight%}

Now we can calculate the fog color based on the fragment's location in 3D space. First, we calculate the Euclidean distance \\(E()\\) between the camera eye \\(C_e\\) and a 3D vertex \\(V_i\\):

\\[ d = E(C_e, V_i) \\]

We use the distance and plug it into our linear function for a fog to get \\(\alpha\\) coefficient. The \\(\alpha\\) coefficient will then be used in GLSL's `mix(x, y, alpha)` function in a manner \\( x(1-\alpha) + y\alpha \\). We derive \\(\alpha\\) by:

\\[ \alpha = 1 - \frac{F_{max} - d}{F_{max} - F_{min}}\\]

where \\(F_{min}\\) and \\(F_{max}\\) are the minimum and muximum distances within which the fog gradient exists. I.e., beyong the minimum distance the geometry or its part will be totally visible, while beyong the maximum distance the geometry will not be visible anymore. 

For simplicity, we set up the fog thresholds inside the fragment shader; but they can also be passed as uniforms. The snippet for the fragment shader is thus:

{%highlight cpp%}
#version 330

uniform vec4 CameraEye;
uniform vec4 FogColor;

in VertexData{
    vec4 mColor;
    vec4 mVertex;
} VertexIn;

float getFogFactor(float d)
{
    const float FogMax = 20.0;
    const float FogMin = 10.0;

    if (d>=FogMax) return 1;
    if (d<=FogMin) return 0;

    return 1 - (FogMax - d) / (FogMax - FogMin);
}

void main(void)
{
    vec4 V = VertexIn.mVertex;
    float d = distance(CameraEye, V);
    float alpha = getFogFactor(d);

    gl_FragColor = mix(VertexIn.mColor, FogColor, alpha);
}
{%endhighlight%}

# Screenshots

Here are some screenshots of the result fog imitation:

![Fog imitation]({{ site.url }}{{ site.baseurl }}/assets/images/osg-shader-3dlines/fog.png)
{: .align-center}

Note: the same scene is displayed but at different distances from the camera point (zoom-out operator performed).

# Code snippet

Check a bit more complex example - [shader-3dcurve](https://github.com/vicrucann/shader-3dcurve). The fogging effect is incorporated into the fragment shader of curve shader. 

