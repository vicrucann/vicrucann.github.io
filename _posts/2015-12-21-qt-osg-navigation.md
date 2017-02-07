---
title: "Step-by-step on how to set up your CMake, Qt and OSG in draw-on-demand mode (Part 2)"
categories: 
    - Tutorials
excerpt: Demonstration on how to trigger OpenSceneGraph events from a Qt widget on example of mouse pressed button.
tags: 
    - OpenSceneGraph 
    - Qt
---

{% include toc %}

# Problem statement

This tutorial is a continuation of [Part 1](http://vicrucann.github.io/tutorials/cmake-qt-osg-1/) where we provided a step-by-step on how to set up Qt with OpenSceneGraph (OSG) in on-demand redraw. 

As a result, the previous tutorial provided a Qt-based widget which was able to display a content of an OSG scene. Although we associated the mouse manipulator with the OSG scene which is responsible for camera manipulations, it was not possible to manipulate the camera view because we did not provide any connection between the Qt mouse events and OSG mouse events. And that is what this tutorial focuses on.

# How to trigger `osgGA::EventQueue` from `QMouseEvent` 

At first, we have to include the necessary Qt mouse events header files:

{% highlight cpp %}
#include <QMouseEvent>
#include <QWheelEvent>
{% endhighlight %}

The next step is to define what events of `QOpenGLWidget` are needed to be redefined. We can find the list of all methods of the widget at the Qt's [manual page](http://doc.qt.io/qt-5/qopenglwidget-members.html). For our case we will only need:

* `mousePressEvent()`
* `mouseReleaseEvent()`
* `mouseMoveEvent()`
* `wheelEvent()`
* `event()` - inherited from `QWidget`. It needs to be re-implemented so that to insure re-draw is performed after every user interaction.

We start from `event` method which calls an `update()` function every time there is an event from user:

{% highlight cpp %}
virtual bool event(QEvent* event)
{
      bool handled = QOpenGLWidget::event(event);
      this->update();
      return handled;
}
{% endhighlight %}

Now we can re-define other Qt mouse events. The main idea is to obtain mouse parameters (for example, coordinates or pressed mouse button code) and use them to trigger OSG mouse events. 

How to do it? Here we will get a help from `getEventQueue()` method (we showed how to implement it in [Part 1](http://vicrucann.github.io/tutorials/cmake-qt-osg-1/) ) which returns an **event queue** from the used **graphics window**. In our case the **event queue** serves as a collector and adaptor of window events. If we look at the [source code
of `osgGA::EventQueue`](https://github.com/openscenegraph/osg/blob/master/include/osgGA/EventQueue), we can see what types of events can be triggered in OSG, and also what parameters they take. 

As an example, let's take the `mouseButtonPress(float x, float y, unsigned int button)`. In order to trigger a **mouse button pressed** event, we have to provide mouse coordinates, as well as a `button` code.

From the [EventQueue](https://github.com/openscenegraph/OpenSceneGraph/blob/master/include/osgGA/EventQueue) file on the mouse button press event:

>  Method for adapting mouse button pressed events, placing this event on the back of the event queue. Button numbering is 1 for left mouse button, 2 for middle, 3 for right.

So, we only have to pass the right button code. To trigger the `mouseButtonPress` from Qt's `mousePressEvent`:

{% highlight cpp %}
virtual void mousePressEvent(QMouseEvent* event)
  {
      unsigned int button = 0;
      switch (event->button()){
      case Qt::LeftButton:
          button = 1;
          break;
      case Qt::MiddleButton:
          button = 2;
          break;
      case Qt::RightButton:
          button = 3;
          break;
      default:
          break;
      }
      this->getEventQueue()->mouseButtonPress(event->x(), event->y(), button);
  }
{% endhighlight %}

Similarly, we have to re-define the **mouse release** event:

{% highlight cpp %} 
virtual void mouseReleaseEvent(QMouseEvent* event)
  {
      unsigned int button = 0;
      switch (event->button()){
      case Qt::LeftButton:
          button = 1;
          break;
      case Qt::MiddleButton:
          button = 2;
          break;
      case Qt::RightButton:
          button = 3;
          break;
      default:
          break;
      }
      this->getEventQueue()->mouseButtonRelease(event->x(), event->y(), button);
  }
{% endhighlight %}

Same manner we can also re-define the **mouse wheel** event. The only difference between the mouse click events and the scroll event, we have to pass the `motion` variable of the type `osgGA::GUIEVentAdapter::ScrollingMotion`, which is an`enum` type of the file [`GUIEventAdapter`](https://github.com/openscenegraph/OpenSceneGraph/blob/master/include/osgGA/GUIEventAdapter). For the simplest mouse moves, we can define whether the mouse scroll was performed `UP` or `DOWN`. The code would look as follows:

{%highlight cpp%}
virtual void wheelEvent(QWheelEvent* event)
  {
      int delta = event->delta();
      osgGA::GUIEventAdapter::ScrollingMotion motion = delta > 0 ?
                  osgGA::GUIEventAdapter::SCROLL_UP : osgGA::GUIEventAdapter::SCROLL_DOWN;
      this->getEventQueue()->mouseScroll(motion);
  }
{%endhighlight%}

After all the event are re-defined, our sample scene (cylinder) can be manipulated by using right and left buttons of the mouse, as well as the wheel. The aforementioned manipulations allow the camera pose change by rotation, pan and zoom.

# Conclusion

This tutorial concludes the minimal CMake, Qt and OSG set up when the scene is rendered without a timer, **on-demand** mode. The second part concentrated mainly on how to use Qt events in order to trigger OSG events, and an example of mouse events (press, release, move) were presented. 

# Codes

[View Code](https://github.com/vicrucann/QtOSG-hello){: .btn--inverse}

You can find the code for this tutorial (both parts 1 and 2) on my [github repository](https://github.com/vicrucann/QtOSG-hello). Note, the presented code also contains parts of code related to [high DPI scaling](http://vicrucann.github.io/tutorials/osg-qt-high-dpi/) which can be easily omitted.

 
