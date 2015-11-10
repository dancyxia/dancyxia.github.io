---
layout: post
title: Linux Development - Debug Tips
category: linux-dev
tags: [linuxdev]
---

### Use gperftool

perftool is used by google for heap profiling and CPU profiling. I only use its heap check function to detect the memory leak. 

Step 1: build perftool
---------------------------------
prerequisite:

Use apt-get install autoconf and libtool

If you are using a 64-bit box, make sure to install [libunwind](http://download.savannah.gnu.org/releases/libunwind/libunwind-0.99-beta.tar.gz) before the build. This is to replace the glibc built-in stack unwinder, which might cause the deadlock.

Build by following this process:

./configure CC=c89 CFLAGS=-O2 LIBS=-lposix
make
make check
make install


Step 2: Configure your Makefile
--------------------------------------

put libtcmalloc and libprofiler to  LDFLAGS as follows:

LDFLAGS = -L/usr/local/lib  -ltcmalloc -lprofiler


Step 3: Run the application
--------------------------------------
run the application test by adding HEAPCHECK=minimal/normal...

HEAPCHECK=normal ./test

If there's a leak, use following command to find more information about the leaks

pprof ./test "/tmp/test.30247._main_-end.heap" --inuse_objects --lines --heapcheck --edgefraction=1e-10 --nodefraction=1e-10 --text


### Use Valgrind


I made another job interview today. Another terrible experience(ㄒoㄒ)

OK...Continue the calendar control...

As I mentioned in last blog, month view consists of a set of custom viewgroup - week view. Today I'd introduce how WeekView is developed.

### Measure ViewGroup

Customizing a ViewGroup is similar to customizing a view. I have already introduced how to define and use the custom attributes, so here I only introduce how to do the measurement and layout.

ViewGroup is responsible for measuring themselves and telling all its children to measure themselves. In our Calendar Control, the ViewGroup(WeekView) consists of seven DayCellView. The height of the row depends on the height of ViewPager and the configuration of the padding width for each day cell. The width of a cell is calculated by dividing the row width with 7. Note we use MeasureSpec.EXACTLY to measure the width and height of the cell, which means each child should be measured to exactly the size it is given.

{% highlight java linenos %}
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        View pager =getRootView().findViewById(R.id.listview);
        if (pager == null) {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
            return;
        }
        final int childCount = this.getChildCount();

        int rowWidth = MeasureSpec.getSize(widthMeasureSpec);
		int cellWidth = rowWidth/childCount;

        int minHeight = pager.getHeight()/WeekListAdapter.maxRow;
        int paddingHeight = (int)getContext().getResources().getDimension(R.dimen.padding)<<1;
        int rowHeight = Math.min(type == LabelType.HEADER? (int)getContext().getResources().getDimension(R.dimen.datelabelsize)+paddingHeight : (int)getContext().getResources().getDimension(R.dimen.daytextsize)+paddingHeight, minHeight);
		int cellWidthMeasureSpec = MeasureSpec.makeMeasureSpec(cellWidth, MeasureSpec.EXACTLY);
		int cellHeightMeasureSpec = MeasureSpec.makeMeasureSpec(rowHeight, MeasureSpec.EXACTLY);
		View child = null;
		for (int i=0; i<childCount; i++) {
			child = this.getChildAt(i);
			if(child.getVisibility() != View.GONE) {
				child.measure(cellWidthMeasureSpec, cellHeightMeasureSpec);
			}
		}
    	this.setMeasuredDimension(rowWidth, rowHeight);
	}
{% endhighlight %}

### Layout ViewGroup

After the ViewGroup is measured, we need to set the bounds for each child. 

{% highlight java linenos %}
	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
		final int childCount = this.getChildCount();
        int left = l;
		for (int i=0; i<childCount; i++) {
			final View child = this.getChildAt(i);
			if (child.getVisibility() != View.GONE) {
                child.layout(left, 0, left + child.getMeasuredWidth(), child.getMeasuredHeight());
                left += child.getMeasuredWidth();
			}
		}
	}
{% endhighlight %}

### Draw the ViewGroup

Once the ViewGroup is measured and layout, the last step is to draw the ViewGroup and its children. Two functions can be called for drawing: onDraw and dispatchDraw. OnDraw happens before the children are drawn. dispatchDraw provides a way to draw the view after the children are drawn. You can do the drawing job after calling super.dispatchDraw. 

In our Calendar Control example, the background grid is drawn in the CalendarMonthView. Nothing is required for WeekView. 

{%  highlight java linenos  %}
protected void dispatchDraw(Canvas canvas) {
        super.dispatchDraw(canvas);
        //draw calendar month view grid
        int childCount = getChildCount();

        //1. get the top，left, right and bottom coordination
        ViewGroup weekView = (ViewGroup)this.getChildAt(0);
        if (null == weekView) return;
        int left = weekView.getLeft();
        int top = weekView.getTop();
        int right = weekView.getRight()-dividerWidth;
        int bottom = this.getChildAt(childCount-1).getBottom()-dividerWidth;

        //2. draw horizontal line
        for (int i = 0; i < childCount; i++) {
            canvas.drawLine(left, top, right, top, framePaint);
            top += this.getChildAt(i).getHeight();
        }
        canvas.drawLine(left, bottom, right, bottom, framePaint);

        //3. draw vertical line
        int cellWidth = weekView.getChildAt(0).getWidth();
        top = weekView.getTop();
        for (int i=0; i< 7; i++) {
            canvas.drawLine(left, top, left, bottom, framePaint);
            left += cellWidth;
        }
        //use the view's right position instead of the calculated result because the calculation result might be
        //a little different from the right position.
        canvas.drawLine(right, top, right, bottom, framePaint);

    }
{%   endhighlight %}

That's how we design our calendar control ~

The code is at [here](https://github.com/swingseagull/Calendar-Control)
