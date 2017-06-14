---
title: "Different approaches of tackling high DPI issues for Qt and OpenSceneGraph/OpenGL applications"
categories: 
    - Tutorials
excerpt: Targeted towards high- & low- resolution monitor combination on Windows, and other high-resolution monitor platforms - using scaling by hand for OpenSceneGraph / OpenGL content, or by setting up the DPI awareness for Windows. 
tags: 
    - OpenSceneGraph 
    - OpenGL
    - Qt
---

{% include toc %}

# Context

Since the release of Qt5.6, high DPI support was introduced in Qt. It is possible to set it by: 

{% highlight cpp %}
int main(int argc, char** argv)
{
    // turn on the DPI support**
    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);

    QApplication app(argc, argv);
    // ...
    return app.exec();
}
{% endhighlight %}

While setting up this attribute to `ON` does its best in high DPI support, it still has lots of room for improvement. In particular, these are the problems which I encountered:

* OpenSceneGraph / OpenGL content scaling is not supported, thus we get the graphics content to be scaled down, as well as corresponding mouse coordinates for the events (see comparative figures below).
* Qt icons look smoothed-out like if they were scaled up (even when using `.svg` format).
* Multi-monitor with different DPI: bad awareness, i.e., some elements like buttons scale better than other (i.e. window title) as a result it produces an ugly different-size fonts window (see example figure below).

<figure class="half">
    <img src="/assets/images/osg-qt-high-dpi/normal-dpi.png" alt="image">
    <img src="/assets/images/osg-qt-high-dpi/high-dpi-failed.png" alt="image">
    <figcaption>Left: DPI support is off; right: DPI support is on. Note the OpenGL content scaling issue on the right. </figcaption>
</figure>

<figure class="half">
    <img src="/assets/images/osg-qt-high-dpi/high-dpi-gui.png" alt="image">
    <img src="/assets/images/osg-qt-high-dpi/normal-dpi-gui.png" alt="image">
    <figcaption>DPI support is ON for multi-monitor and different DPI; left: main high DPI monitor; right: second low DPI monitor. While the problem with the OpenGL content scaling persists for high DPI monitor on the left, the font is not scaled correctly on low resolution monitor for different GUI elements such as window title. </figcaption>
</figure>


For my application it was evident that I had to find another solutions depending on the target platform, so in my code I made sure to turn off the Qt support for high DPI:

{% highlight cpp %}
// turn the DPI support off
QApplication::setAttribute(Qt::AA_DisableHighDpiScaling);
{% endhighlight %}

**Tutorial content**: this post will introduce couple ideas that can be used on how to deal with the high DPI issues. It is not intended to be thorough, but rather to provide different directions which can be chosen depending on your platform, application, monitor set up, etc.

**OpenGL remark:** any code snippets that I will be using are of either Qt or OpenSceneGraph. In this tutorial I am not talking about OpenGL directly, but by using OpenSceneGraph library. Since OSG is an OpenGL wrapper, the covered principles can be applied to OpenGL the same way.
{: .notice--info}

**Reference:** this article is a continuation of my [stackoverflow question](http://stackoverflow.com/questions/37303941/qt5-6-high-dpi-support-and-opengl-openscenegraph).
{: .notice--info}

# Windows platform quick fix 

## Desktop Qt and OpenSceneGraph / OpenGL applications

This is a simple solution that works very well if your main target application is Windows platform. The Windows version must be Vista or higher. The MSDN functions is called [`SetProcessDPIAware()`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms633543%28v=vs.85%29.aspx). 

Assuming you are given some Qt (plus OSG/OpenGl) application, you can achieve nice high DPI scaling and multi-monitor awareness at the same time. 

This is the code snippet of the `main.cpp` which uses the MSDN function:

{% highlight cpp %}
#include <Windows.h>

int main(int argc, char** argv)
{
#ifdef Q_OS_WIN
    SetProcessDPIAware(); // call before the main event loop
#endif // Q_OS_WIN 

#if QT_VERSION >= QT_VERSION_CHECK(5,6,0)
    QApplication::setAttribute(Qt::AA_DisableHighDpiScaling);
#else
    qputenv("QT_DEVICE_PIXEL_RATIO", QByteArray("1"));
#endif // QT_VERSION

    QApplication app(argc, argv);
    // ...
    return app.exec();
}
{% endhighlight %}

As a result you obtain a GUI that looks the same no matter what monitor density is.

## Full screen OpenSceneGraph applications

For a pure OpenSceneGraph full screen application (when dealing with multi-monitor system), you have to specify that viewer should be mapped to only a single screen. It can be especially useful when trying to run OSG example on high DPI and/on multi-monitor multi-DPI screens. The below snippet provides an example of a simple OSG application:

{% highlight cpp %}
#include <Windows.h>

int main(int argc, char** argv)
{
    ::SetProcessDPIAware();

    osg::ref_ptr<osg::Node> root = osgDB::readNodeFile("cow.osgt");
    osgViewer::Viewer viewer;
    viewer.setSceneData(root.get());

    viewer.setUpViewOnSingleScreen(0);
    return viewer.run();
}
{% endhighlight %}

Setting up those parameters helped to resolve visual artefacts and crashing when running OSG examples on a two monitor system (high DPI laptop and normal DPI screen) on Windows platform. However, you would have to edit the `main.cpp` file and insert the code like in the above snippet for each example before running them.

## Summary

**Pros**: using `SetProcessDPIAware` (or its alternative `SetProcessDpiAwareness()`) is a quick hack to instantly solve high DPI and multi DPI issues for the modern Windows applications.

**Cons**: it is only available on Windows platform. 

**Remark:** when running the application Qt might complain in the log: `SetProcessDpiAwareness failed: "COM error 0xffffffff80070005  (Unknown error 0x0ffffffff80070005)"`. 
{: .notice--info}

# Manual scaling of OpenSceneGraph / OpenGL content

This sections is rather experimental and I am not sure how practical it is. Assuming that, for some reason, you decide to turn on the high DPI support `ON` using Qt (or it is turned on automatically by your operating system), this part of tutoral provides a general guidance on how to deal with scaled-down OSG/OpenGL content. 

## OSG / OpenGL parameters to scale up

Assume you are given a `QOpenGLWidget` which contains some OSG/OpenGL scene. As an example, I could refer to one of my previous tutorials where I set up [Qt and OpenSceneGraph application in draw-on-demand mode](http://vicrucann.github.io/tutorials/cmake-qt-osg-1/). I will try not to tie very much to that tutorial so that this post remains rather generic.

One of the reasons why OpenGL content of the `QOpenGLWidget` is not scaled up when we set the high DPI support to `ON` is that we have to provide correct scaling inside the `QOpenGLWidget` when passing dimensional parameters or the coordinates of the widget to the OSG/ OpenGL content. These are the examples of dimensions: 

* `QOpenGLWidget::width()` 
* `QOpenGLWidget::height()` 
* and its mouse events - `QMouseEvent::x()` and `QMouseEvent::y()`

To be more specific, assume we re-defined `QOpenGLWidget::resizeGL()` method which also performs the resizing of our OSG / OpenGL content. This how scene resizing is normally performed (this snippet is taken from [Qt + OSG tutorial](http://vicrucann.github.io/tutorials/cmake-qt-osg-1/)) :

{% highlight cpp %}
  virtual void resizeGL( int width, int height ) 
  {
      this->getEventQueue()->windowResize(this->x(), this->y(), width, height);
      _mGraphicsWindow->resized(this->x(), this->y(), width, height);
      osg::Camera* camera = _mViewer->getCamera();
      camera->setViewport(0, 0, this->width(), this->height());
  }
{% endhighlight %}

The sizing we pass to the OSG content is not scaled correctly because of the Qt auto scaling. The same happens to mouse events coordinates: they are not passed correctly. 

**To resolve**: we have to provide some manual scaling for each dimensional parameter. In general, it would be easier to calculate the screen scales for `X` and `Y` directions and apply them to any dimensional parameter which is passed to the OSG/OpenGL content.

## Scale calculation

To calculate the scaling values for `X` and `Y` directions, we need to know some reference values of DPI in those dimensions. The easy working solution is to run your application without the DPI support, get the reference values and then perform the current setting calculation so that to pass them to the `QOpenGLWidget`. The code of the `main()` has to contain:

{%highlight cpp%}
QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
QApplication app(argc, argv);

int x = QApplication::desktop()->physicalDpiX();
int y = QApplication::desktop()->physicalDpiY();
// values 284 and 285 are the reference values
double scaleX = 284.0/double(x);
double scaleY = 285.0/double(y);

QMainWindow window;
QtOSGWidget* widget = new QtOSGWidget(scaleX, scaleY, &window);
// etc.
{% endhighlight %}

**Update:** there is a much easier way to calculate the scaling parameter. So, for all the OpenGL-related parameters such as widgets and events, it is sufficient to use `QApplication::desktop()->devicePixelRatio()` as a scaling value. 
{: .notice--warning}

## Scale usage within `QOpenGLWidget`

Assuming you had passed the `scaleX` and `scaleY` to the `QOpenGLWidget`, you can use them when passing the Qt dimensions such width and height, and mouse positions for events to the OSG / OpenGL content. Following the same example of `resizeGL()`, this is how the method will turn out:

{% highlight cpp %}
  virtual void resizeGL( int width, int height ) 
  {
      this->getEventQueue()->windowResize(this->x()*m_scaleX, this->y() * m_scaleY, width*m_scaleX, height*m_scaleY);
      _mGraphicsWindow->resized(this->x()*m_scaleX, this->y() * m_scaleY, width*m_scaleX, height*m_scaleY);
      osg::Camera* camera = _mViewer->getCamera();
      camera->setViewport(0, 0, this->width()*m_scaleX, this->height()* m_scaleY);
  }
{% endhighlight %}

After we make sure we multiply every Qt dimension that we pass with the obtained scale, our application's content finally runs correctly.

**Update:** since we now calcualte the `scale` using `QApplication::desktop()->devicePixelRatio()`, we will have only one `scale` value which will be applied the same manner for the resizing example above. 
{: .notice--warning}

## Summary

**Pros**: total control of OSG / OpenGL content scaling which can be the only option when target OS provides automatic scaling (think retina display on MacOS).

**Cons**: to resolve the multi-monitor awareness, have to perform the OSG/OpenGL content re-scaling whenever the application is dragged from one screen to another. 

**Remark:** unfortunately, I do not have an opportunity to perform proper tests on other operating systems such as MacOS, that is why this part is more theoretical and only serves as an example of possible solution to the auto DPI scaling.
{: .notice--info}

# Codes

You can check the [minimal Qt + OSG code](https://github.com/vicrucann/QtOSG-hello). Try to switch `QApplication::setAttribute(Qt::AA_DisableHighDpiScaling);` and see how the manual scaling works (I had only tested on Windows, but it based on the recieved user feedback, that seems to work on Mac machines with Retina display as well).

