---
title: "How to customize OpenSceneGraph's line intersector to select line segments"
categories: 
    - Tutorials
excerpt: A tutorial which demonstrates how to re-define your own entity (edge, or line segment) selector based on `osgUtil::LineSegmentIntersector`.
tags: 
    - OpenSceneGraph 
    - 3D geometry 
mathjax: true
---

{% include toc %}

# Problem statement

This tutorial is based on the section *Designing customized intersectors* of Chapter 10, *OpenSceneGraph Cookbook*[^ref_cookbook]. The main difference between this tutorial and the reference material is that we demonstrate how to extend the **point selector** into **line segment selector**.

[^ref_cookbook]: [Chapter 10](http://documents.tips/documents/openscenegraph-cookbook-chapter-10.html) of OpenSceneGraph Cookbook, Rui Wang and Xuelei Qian.

This tutorial is potentially not complete[^next] and aimed to only provide a base of line segment selection, i.e., the line segment selection is only possible by clicking next to the segment's vertices, and not necessary next to the line itself. The selection of the line segment by clicking at any point next to the line will be covered in one of the future tutorials.

[^next]: For completion, go to [part 2](http://vicrucann.github.io/tutorials/osg-stroke-intersector-improved/).

# Line segment class

We define a line segment as two vertices connected by a line (edge). 

![Line segment representation]({{ site.url }}/images/osg-stroke-selector/line-segment.png)
{: .pull-center}

Our line segment class should allow us to set up the coordinates for the segment ends, and also change the segment color, for example, when it is selected by user. It has the following interface:

{% highlight cpp %}
class LineSegment : public osg::Geometry {
public:
    LineSegment();
    void setColor(const osg::Vec4& color);
    void setBeginEnd(float x0, float y0, float x1, float y1);

protected:
    ~LineSegment(){}
};
{% endhighlight %}

Check the implementation details in the accompanying code.

# User-defined intersector for selection of line segments

In order to avoid ambiguity with names (the base line intersector is already named `osgUtil::LineSegmentIntersector` since it searches intersection between line segment cast by mouse and geometries), we will call our customized intersector as `StrokeIntersector`. The `StrokeIntersector` means finding intersections between line segment cast by mouse and stroke (or line segment) drawn by user on a scene.

For our task, we can follow the same steps as in tutorial of Chapter 10 of *OpenSceneGraph Cookbook*[^ref_cookbook]. First, we define the `StrokeIntersector` API. We have to override the constructors, method `clone()` and the most important method `intersect()`:

{% highlight cpp %}
class StrokeIntersector : public osgUtil::LineSegmentIntersector
{
public:
    StrokeIntersector();
    StrokeIntersector(const osg::Vec3& start, const osg::Vec3& end);
    StrokeIntersector(CoordinateFrame cf, double x, double y);

    void setOffset(float o);
    float getOffset() const;

    virtual Intersector* clone( osgUtil::IntersectionVisitor& iv );
    virtual void intersect( osgUtil::IntersectionVisitor& iv, osg::Drawable* drawable );

protected:
    virtual ~StrokeIntersector(){}

private:
    float m_offset;
};
{% endhighlight %}

The constructors and method `clone()` are straightforward to implement. For the `intersect()`, we only have to process those `osg::Geometry`s that are of the type `LineSegment` (for this we use `dynamic_cast`). All the rest of the implementation remains the same.

{% highlight cpp %}
void StrokeIntersector::intersect(osgUtil::IntersectionVisitor& iv, osg::Drawable* drawable)
{
    osg::BoundingBox bb = drawable->getBoundingBox();
    bb.xMin() -= m_offset; bb.xMax() += m_offset;
    bb.yMin() -= m_offset; bb.yMax() += m_offset;
    bb.zMin() -= m_offset; bb.zMax() += m_offset;

    osg::Vec3d s(_start), e(_end);
    if ( !intersectAndClip(s, e, bb) ) return;
    if ( iv.getDoDummyTraversal() ) return;

    LineSegment* geometry = dynamic_cast<LineSegment*>(drawable->asGeometry());
    if ( geometry )
    {
        osg::Vec3Array* vertices = dynamic_cast<osg::Vec3Array*>( geometry->getVertexArray() );
        if ( !vertices )
            return;

        osg::Vec3d dir = e - s;
        double invLength = 1.0 / dir.length();

        for ( unsigned int i=0; i<vertices->size(); ++i )
        {
            double distance =  fabs( (((*vertices)[i] - s)^dir).length() );
            distance *= invLength;
            if ( m_offset<distance ) continue;

            Intersection hit;
            hit.ratio = distance;
            hit.nodePath = iv.getNodePath();
            hit.drawable = drawable;
            hit.matrix = iv.getModelMatrix();
            hit.localIntersectionPoint = (*vertices)[i];
            insertIntersection( hit );
        }
    }
}
{% endhighlight %}

# Line segment selection inside event handler

We use the customized intersector inside a customized event handler. The algorithm is as follows:

* If right mouse button is pressed,
* Initialize `StrokeIntersector` and pass the current camera view to it.
* If the intersector contains any intersections as result,
* Get a drawable result and cast it to the `LineSegment` type.
* If the cast was successful, change the color of that line segment. Also, make sure the previously selected line segment is unselected.

{% highlight cpp %}
virtual bool handle(const osgGA::GUIEventAdapter& ea, osgGA::GUIActionAdapter& aa)
    {
        if (!(ea.getEventType() == osgGA::GUIEventAdapter::RELEASE && ea.getButton()==osgGA::GUIEventAdapter::RIGHT_MOUSE_BUTTON))
            return false;
        osgViewer::View* viewer = dynamic_cast<osgViewer::View*>(&aa);
        if ( viewer )
        {
            StrokeIntersector* intersector = new StrokeIntersector(osgUtil::Intersector::WINDOW, ea.getX(), ea.getY());
            osgUtil::IntersectionVisitor iv(intersector);

            osg::Camera* camera = viewer->getCamera();
            if(!camera) return false;
            camera->accept(iv);

            if(!intersector->containsIntersections()){
                std::cout << "no intersections found" << std::endl;
                return false;
            }
            const StrokeIntersector::Intersection& result = 
                                        *(intersector->getIntersections().begin());
            LineSegment* line = dynamic_cast<LineSegment*>(result.drawable.get());
            if (!line){
                std::cerr << "could not dynamic_cast to LineSegment" << std::endl;
                return false;
            }

            if (m_line.get()) m_line->setColor(osg::Vec4(0,0,0,1));
            m_line = line;
            m_line->setColor(osg::Vec4(1,0,0,1));
        }
        return false;
    }
{% endhighlight %}

# Conclusion

In this brief tutorial we demonstrated how to use point selector example of *OpenSceneGraph Cookbook*[^ref_cookbook] and derive your own line segment selector. This tutorial is not complete since it only allows stroke selection by clicking close to the stroke vertices (\\(x_0\\) and \\(x_1\\) on the figure below). The complete version would mean to perform selection when clicked close to any point \\(x_i\\) on the line segment.

![Line segment selection]({{ site.url }}/images/osg-stroke-selector/selection-complete.png)
{: .pull-center}

For the moment, this part is out-of-scope of this tutorial. However, it is covered in the [second part](http://vicrucann.github.io/tutorials/osg-stroke-intersector-improved/) of the tutorial.

# Codes

Check a bit more complex example - [OSG intersectors](https://github.com/vicrucann/osg-intersectors-example) where three different intersectors are presented: line, point and with a virtual plane (based on [raycast algortihm](http://vicrucann.github.io/tutorials/osg-raycast/)). With [OSG intersectors code](https://github.com/vicrucann/osg-intersectors-example), the fully functional line intersector is used when performing hovering over the wire in order to select it.

