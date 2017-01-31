---
title: "How to obtain OpenGL version from within OpenSceneGraph"
categories: 
    - Tutorials
excerpt: A self content OSG code snippet that demonstrates how to get the OpenGL version with some explanations.
tags: 
    - OpenSceneGraph 
    - OpenGL
    - GLSL
---

{% include toc %}

# Introduction

When writing a shader for one of my projects, I encountered a need to define an OpenGL version and whether GLSL is supported on the used machine. Assume, I have some geometry and there are two ways to render it: simplified version and fancy version. The simplified version uses some default primitive, e.g., `GL_LINE_STRIP_ADJACENCY`. The fancy version requires certain minimal OpenGL version so that to use the shaders I just wrote. 

Since in OpenSceneGraph we normally do not deal with OpenGL commands directly, I had to find a way how to define the supported OpenGL versions through the OSG library.

# Tester class for supported OpenGL version

The OSG examples already have an example that provides an idea how to request certain OpenGL constants. For instance, OsgShaderTerrain example. I took the derived class from `osg::GraphicsOperation` and modified it so that it returned the OpenGL version. Below is an example of how such tester class is implemented:

{%highlight cpp%}
class TestSupportOperation : public osg::GraphicsOperation
{
public:
    TestSupportOperation()
        : osg::Referenced(true)
        , osg::GraphicsOperation("TestSupportOperation", false)
        , m_supported(true)
        , m_errorMsg()
        , m_version(0.0)
    {}

    virtual void operator() (osg::GraphicsContext* gc)
    {
        OpenThreads::ScopedLock<OpenThreads::Mutex> lock(m_mutex);
        osg::GLExtensions* gl2ext = gc->getState()->get<osg::GLExtensions>();

        if( gl2ext ){

            if( !gl2ext->isGlslSupported )
            {
            m_supported = false;
            m_errorMsg = "ERROR: GLSL not supported by OpenGL driver.";
            }
            else
                m_version = gl2ext->glVersion;
        }
        else{
            m_supported = false;
            m_errorMsg = "ERROR: GLSL not supported.";
        }
    }

    OpenThreads::Mutex  m_mutex;
    bool                m_supported;
    std::string         m_errorMsg;
    float               m_version;
};
{%endhighlight%}

Now, when using the `TestSupportOperation` class we can easily obtain the OpenGL version by refering to the class' public variable: `tester->m_version`.

A very simplified case usage (empty scene) is presented below:

{%highlight cpp%}
int main(int, char**)
{
    osgViewer::Viewer viewer;
    viewer.setUpViewInWindow(100,100,1024,960);

    // openGL version:
    osg::ref_ptr<TestSupportOperation> tester = new TestSupportOperation;
    viewer.setRealizeOperation(tester.get());
    viewer.realize();

    if (tester->m_supported)
        std::cout << "GLVersion=" << tester->m_version << std::endl;
    else
        std::cout << tester->m_errorMsg << std::endl;

    return viewer.run();
}
{%endhighlight%}
