---
title: "QTreeWidget: signal emission on internal moves of children within a set of parents"
categories: 
    - Tutorials
excerpt: How to get notification and then send a signal on drag-and-drop event of QTreeWidget.
tags: 
    - Qt 
---

{% include toc %}

# Initial setup

For this problem we are given `QTreeWidget` which consists of set of parents which can have a set of children. For simplicity we set the children to behave like leafs which means they cannot have their own children. The figure below provides an example of such tree structure:

![QTreeWidget example structure]({{ site.url }}/images/qtreewidget-drag/qtree-example.png)
{: .pull-center}

Assume, we have implemented the rules for drag and drop of the children, for example, we can only drag one child at a time to another parent; and we cannot drag the parent at all.

# Problem statement

For the given setup we want to be able to emit signal whenever the drag and drop action occurs. By default Qt does not provide such a signal. While drag and drop is carried out by Qt within the `QTreeWidget`, the signal functionality might be useful if we also want to work with certain external data.

As an example, in my case it was OpenSceneGraph data structure and my task was to perform scene graph re-arrangement whenever drag-and-drop occurred. 

The result output parameters should contain the next information:

* **parent** top-level item from where the children are moved
* **start** and **end** indexes of child(-ren) before the move; within the parent item
* **destination** top-level item to where the children are moved
* **row** index where the children were inserted within the destination item

# Solution

In order to obtain and pass the required parameters on drag and drop within `QTreeWidget`, we have to re-implements its protected method `dropEvent`. Here is the code sample of how to do it:

{% highlight cpp %}
void TreeWidget::dropEvent(QDropEvent *event)
{
    QList< QTreeWidgetItem* > kids = this->selectedItems();

    /* row number before the drag - initial position */
    if (kids.size() == 0) return;
    int start = this->indexFromItem(kids.at(0)).row();
    int end = start; // assume only 1 kid can be dragged
    QTreeWidgetItem* parent = kids.at(0)->parent();

    /* perform the default implementation */
    QTreeWidget::dropEvent(event);

    /* get new index */
    int row = this->indexFromItem(kids.at(0)).row();
    QTreeWidgetItem* destination = kids.at(0)->parent();

    if (!parent || !destination){
        event->setDropAction(Qt::IgnoreAction);
        return;
    }

    /* emit signal about the move */
    emit childDragedAndDroped(parent, start, end, destination, row);
}
{% endhighlight %}

If necessary we can always obtain the indices of both `parent` and `destination` by using `QTreeWidget::indexOfTopLevelItem(QTreeWidgetItem*)`. 

The sample above provides an example when only one child can be dragged. In case for the multiple children drag (consequitive order), we would have to assign `end = this->indexFromItem(kids.at(kids.size()-1)).row();`.
