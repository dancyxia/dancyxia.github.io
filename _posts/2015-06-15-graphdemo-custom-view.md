---
layout: post
title: Make a custom view - GraphView
category: android-app-graphdemo
tags: [algorithm, android]
---
### Design consideration

#### Cusom View

Considering that the Graph generator is the base for all other Graph related algorithm demonstration, it should be decoupled from the embedding app. A custom view and a independent fragment container would make it more independent and easier to be integrated with other components.

Generally, to create a custom view, we need to pay attention to four functions: custom attributes, measurement, draw and event handling. So let's start from  customizing attributes for our GraphView.

##### Custom Attributes

To make your own attributes work, we would go through following process: 

- **Define** the attributes 

   We create a new xml file: *attrs_graph_view.xml* and place it to res/values/ folder. The attribute file defines our custom attributes. 

{% highlight text linenos %}

<resources>
    <declare-styleable name="GraphView">
        <attr name="nodeShape" format="reference" />
        <attr name="edgeColor" format="color" />
        <attr name="textColor" format="color" />
    </declare-styleable>
</resources>
{% endhighlight %}

Attributes are defined inside a *declare-styleable* element. Here we only define three custom attributes, nodeShape is with reference format. It references to a drawable. edgeColor and textColor are both defined as *color* type. I found a very good explanation on how to define attributes at [stackoverflow](http://stackoverflow.com/a/3441986/1411938)

- **Set** the values for the attributes 
Then we can specify the values for these custom attributes in your own layout xml. 

{% highlight text linenos %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent" android:layout_height="match_parent">

    <com.dancy.graphdemo.GraphView{
        android:id="@+id/graphview"
        android:background="#fff"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingLeft="20dp"
        android:paddingBottom="40dp"
        app:edgeColor="#0000ff"
        app:textColor="#ffffff"
        app:nodeShape="@drawable/node_shape" />
</FrameLayout>
{% endhighlight %}

To apply your own attributes you need to create an alias for your custom view namespace and use it for the attributes you defined. The namespace is like this:  http://schemas.android.com/apk/res/[your package name]. But you can also use *res-auto* instead. Here we use *app* as the namespace alias, but you can use any alias.

- **Obtain** the values when instantiate the view.
The last thing is using the attributes in the view code. 

{% highlight java linenos %}
        Drawable nodeDrawable = a.getDrawable(R.styleable.GraphView_nodeShape);
        edgeColor = a.getColor(R.styleable.GraphView_edgeColor, Color.BLUE);
        textColor = a.getColor(R.styleable.GraphView_textColor, Color.WHITE);
        nodeSize = nodeDrawable.getIntrinsicWidth();

{% endhighlight %}

#### Measurement

Usually we also need to rewrite the onMeasure() function and onLayout() function to adjust the view size and layout. This is extremely important when you write a custom ViewGroup. However, for our GraphView, we don't need to override it.

#### Draw the view

Now it's time to draw the graph. The drawing is implemented in function onDraw(). Here we need to generate several nodes and place them randomly on the view. The node shape, node size and node colour is defined in node_shape.xml, which located in res/drawable folder.  Before drawing, a pre-draw node bitmap is generated. And this bitmap is used to draw all nodes on the view.

{% highlight java linenos %}
   @Override
protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        if (graph == null)
            return;

        if (contentWidth == 0 && contentHeight == 0) {
            paddingLeft = getPaddingLeft();
            paddingTop = getPaddingTop();
            paddingRight = getPaddingRight();
            paddingBottom = getPaddingBottom();

            contentWidth = getWidth() - paddingLeft - paddingRight;
            contentHeight = getHeight() - paddingTop - paddingBottom;
        }

        Paint pt = new Paint();
        pt.setColor(edgeColor);
        //first time draw nodes
        if (!bGraphDataReady) {
            Random rand = new Random();
            rand.setSeed(Calendar.getInstance().getTime().getTime());

            // allocations per draw cycle.
            for (int i = 0; i < graph.V(); i++) {
                int x = rand.nextInt(contentWidth - nodeSize);
                int y = rand.nextInt(contentHeight - nodeSize);

                insertNodeAt(new Rect(x, y, x + nodeSize - 1, y + nodeSize - 1), i);
                canvas.drawBitmap(bm, x, y, pt);
                String value = String.valueOf(i);
                Rect textRect = new Rect();
                textPt.getTextBounds(value, 0, value.length(), textRect);
                canvas.drawText(value, x+((nodeSize-textRect.width())>>1), y+((nodeSize+textRect.height())>>1), textPt);
            }
            bGraphDataReady = true;
        } else { //redraw after view is updated
            for (int node : sortedNode) {
                canvas.drawBitmap(bm, nodeList[node].left, nodeList[node].top, pt);
                String value = String.valueOf(node);
                Rect textRect = new Rect();
                textPt.getTextBounds(value, 0, value.length(), textRect);
                canvas.drawText(value, nodeList[node].left + ((nodeSize - textRect.width()) >> 1), nodeList[node].top + ((nodeSize + textRect.height()) >> 1), textPt);
                for (Graph.Edge edge : graph.getEdges(node)) {
                    canvas.drawLine(nodeList[node].centerX(), nodeList[node].centerY(), nodeList[edge.end].centerX(), nodeList[edge.end].centerY(), pt);
                }
            }

            //this is to draw the dragging line
            if (runningNode != null) {
                canvas.drawLine(nodeList[touchNode].centerX(), nodeList[touchNode].centerY(), runningNode.centerX(), runningNode.centerY(), pt);
            }
        }
    }

{% endhighlight %}


#### Event handling

Usually we need to capture the touch event and handle it in custom view. Here we override onTouchEvent to implement the node dragging and edge adding.

{% highlight java linenos %}
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        super.onTouchEvent(event);
            if (operationID == OPERATION.LINK_NODE) {
                return onTouchEventForLink(event);
            } else {
                return onTouchEventForMove(event);
            }
    }

    private boolean onTouchEventForMove(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                if (isDragging)
                    return true;
                prepareDraggingForNodeMove(event);
                return true;
            case MotionEvent.ACTION_MOVE:
                if (!isDragging)
                    prepareDraggingForNodeMove(event);
                else {
                    long now = System.currentTimeMillis();
                    if (isDragging && now - lastTimeStamp > 60) {
                        int cx = (int) event.getX();
                        int cy = (int) event.getY();
                        Rect rect = new Rect(nodeList[touchNode]);
                        nodeList[touchNode].offset(cx - lastPos.x, cy - lastPos.y);
                        touchedNodeIdx = GraphViewHelper.resortNodeFor(sortedNode, nodeList, touchedNodeIdx);
                        rect.union(nodeList[touchNode]);
                        for (Graph.Edge e : graph.getEdges(touchNode)) {
                            rect.union(nodeList[e.end]);
                        }
                        lastPos.set(cx, cy);
                        invalidate(rect);
                        lastTimeStamp = now;
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                touchNode = -1;
                touchedNodeIdx = -1;
                isDragging = false;
                break;
            default:
                break;
        }
        return true;
    }

    private void prepareDraggingForNodeMove(MotionEvent event){
        touchedNodeIdx = GraphViewHelper.getTouchNode(sortedNode, nodeList, new Point((int)event.getX(), (int)event.getY()));
        if (touchedNodeIdx > -1) {
            touchNode = sortedNode[touchedNodeIdx];
            lastPos = new Point((int)event.getX(), (int)event.getY());
            lastTimeStamp = System.currentTimeMillis();
            isDragging = true;
        }
    }
	
{% endhighlight %}

See figure below for the final effect

![placeholder](/images/graphview/graph_view.png)



#### Thanks To [Icons4android](http://www.icons4android.com/) for providing the amazing icons