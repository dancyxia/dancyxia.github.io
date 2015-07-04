---
layout: post
title: Calendar Control Widget - Month View Transition
category: android development
tags: [calendarcontrol]
---

Finished a online technical quiz. I failed to complete it on time(Only had the coding done. Don't have enough time for debug. And at night I recalled I made some critical mistakes in the code :() I don't expect any further message from the company(how sad!). However, I still tried redoing the quiz and made it run. Will write a separate blog for it later.

### Overview
Now blog and github time again. I reviewed the calendar widget I wrote two years ago and decided to write some blogs before archiving it. 

The purpose of the widget is to provide a month view calendar widget which then can be used in various applications. The month can be switched back and forth by sliding left or right. The changing event of the month(or day) can be observed by other views and then cause the change of their own.

I decided to use ViewPager for the smoothly month page switching. Considering that a month view actually is a list of week view, so I use the list view to represent it. I use a custom view group which includes the day cell view to handle the week view.

Here's the effect of the calendar control widget.

![placeholder](/images/calendarcontrol/calendar_portrait_view.gif)

### ViewPager

ViewPager is powerful for the screen transition. So I use it to hold the Month page. Let's first have a look at the layout.

{% gist /swingseagull/cf50b887516b7db4cb49 calendar_month_pager.xml %}

Element ViewPager is contained in the layout document to indicates the placeholder of the pager to be transited.

And then create an adapter extends FragmentStatePagerAdapter. The adapter needs to implement function getCount() and getItem(int idx). For Calendar month view, only three pagers are required: the previous month page(0 as the index), current month page(1 as the index) and the next month page(2 as the index). So it always returns 3 for function getCount().  By the way, don't forget to set the adapter to the view.

{% highlight java linenos %}
	private static class MonthPagerAdapter extends FragmentPagerAdapter {

		public MonthPagerAdapter(FragmentManager fm) {
            super(fm);
		}

		@Override
		public Fragment getItem(int idx) {
            MonthFragment fragment = MonthFragment.newInstance(idx);
            return fragment;
		}

		@Override
		public int getCount() {
			return 3;
		}
		
	}
{% endhighlight %}

As getItem returns a Fragment object. So the pager should be managed by Fragment. Here is the fragment class.

{% highlight java linenos %}

public class MonthFragment extends Fragment {
    private MonthChangeListener monthChangeListener;

    public interface MonthChangeListener {
        public void onMonthChanged(Calendar thisMonth);
    }

    public static MonthFragment newInstance(int index) {
        MonthFragment newFragment = new MonthFragment();
        Bundle args = new Bundle();
        args.putInt("index", index);
        newFragment.setArguments(args);
        return newFragment;
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            this.monthChangeListener = (MonthChangeListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString() + " must implement MonthChangeListener");
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        if (container == null)
            return null;
        ViewGroup view = (ViewGroup)inflater.inflate(R.layout.activity_main, null);
        updateView(view);
        return view;
    }

{% endhighlight %}

Finally, to make the pager's moving loop work as calendar month transition, we need to update the month data for each pager view whenever there's the page transit event. I made it by implementing the Viewpager.OnPageChangeListener. When the pager is switched to left, the position in function onPageSelected() would be 0. We record this value. In function onPageScrollStateChanged, we reset currentMonth with this value.  And we do the same for the operation of switching to the right side. 

{%  highlight java linenos %}
		pager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {

            @Override
            public void onPageSelected(int position) {
                currentIndex = position;
            }

            @Override
            public void onPageScrolled(int arg0, float arg1, int arg2) {}

            @Override
            public void onPageScrollStateChanged(int state) {
                if (state == ViewPager.SCROLL_STATE_IDLE) {
                    currentMonth.add(Calendar.MONTH, currentIndex - 1);
                    pager.setCurrentItem(1, false);//this does the trick
                    for (int i = 0; i < 3; i++) {
                        MonthFragment fragment = (MonthFragment) ((MonthPagerAdapter) pager.getAdapter()).instantiateItem(pager, i);
                        ViewGroup rootView = (ViewGroup) fragment.getView();
                        if (rootView != null) {
                            fragment.updateView(rootView);
                        }
                    }
                }
            }
        });

{%  endhighlight %}

![placeholder](/images/calendarcontrol/calendar_portrait_view.gif)