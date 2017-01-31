---
title: "How to simulate mouse click within QTestLib on items and elements of QTreeWidget/QListWidget"
categories: 
    - Tutorials
excerpt: Given a widget of type QTreeWidget/QListWidget, we want to be able to simulate a mouse click event on one of the items, or a button within one of the items. 
tags: 
    - Qt
---

{% include toc %}

# Context

While writing some unit tests for my program, I encountered a problem of how to simulate the mouse event within the Qt widgets such as `QListWidget` and `QTreeWidget`. In particular, I wanted to trigger two types of events:

* mouse click on an item of my widget
* mouse click on a button within the item of my widget

This short tutorial provides the code snippets that helped me to solve the problem. From the start, we will rely on the Qt class that is able to test for emitted signals - [QSignalSpy](http://doc.qt.io/qt-5/qsignalspy.html). It will help us to define whether the event happened or not.

# Event within an item of the list widget

Assume we are given a `QListWidget* listWidget;` and we want to test if the click was performed on the first list item. The below code snippet shows how to achieve it.

{%highlight cpp%}
QSignalSpy spy(listWidget, SIGNAL(clicked(QModelIndex)));

// obtain the 0th list item you want to click on
QListWidgetItem* item = listWidget->item(0);
QVERIFY(item);

// obtain the rectangular coordinates of the item
QRect rect = listWidget->visualItemRect(item);

// imitate the click on the 0th item
QTest::mouseClick(listWidget->viewport(), Qt::LeftButton, 0, rect.center());

// check the signal was indeed emitted, one time
QCOMPARE(spy.count(), 1);

// check the type of passed argument
QList<QVariant> args = spy.takeFirst();
QVERIFY(args.at(0).type() == QVariant::ModelIndex);
{%endhighlight%}

# Event within an item's button of the tree widget

![QTreeWidget example]({{ site.url }}/images/treewidget.png)
{: .pull-center}

Assume we are given a `QTreeWidget* treeWidget;`, which has its own delegate where it is defined how to place buttons within it, and also how to process mouse events, for example, if the user presses on one of the buttons. We want to be able to simulate those mouse events and check if the necessary signals are emitted. The below snippet provides the basic idea on how to achieve it. 

{%highlight cpp%}
QSignalSpy spy(treeWidget->getTreeWidgetDelegate(), SIGNAL(clickedButton(int)));

// given a treeWidget with some items in it, obtain the 0th item
QTreeWidgetItem* item = treeWidget->topLevelItem(0);
QVERIFY(item);
QVERIFY(item->count() > 0);
treeWidget->expandAll();

// obtain an item's 0th child where we want to trigger a button press event
QTreeWidgetItem* child = item->child(0);
QVERIFY(child);

// obtain the rectangular coordinates of the child item
QRect rect = treeWidget->visualItemRect(child);
QRect rectButton = treeWidget->getTreeWidgetDelegate()->getButtonRect(rect);

// simulate the mouse click within the button coordinates
QTest::mouseClick(treeWidget->viewport(), Qt::LeftButton, 0, rectButton.center());

// make sure the necessary signal was emitted
QCOMPARE(spy.count(), 1);
{%endhighlight%}

**Note:** I do not provide code reference for the widget's delegate. As it can be seen from the snippet, the delegate contains a method to obtain a button's coordinates. It also contains the signal which is emitted whenever that button is pressed. The `treeWidget` class contains a method that retunrs a pointer on its delegate.
{: .notice--info}

