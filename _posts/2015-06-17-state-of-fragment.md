---
layout: post
title: Maintain the state of fragments
category: android-app-graphdemo
tags: [algorithm, android, fragment]
---

### Fragment Lifecycle

Today I fixed the data lost issue when rotating the device or switching back and forth between applications. I place the custom view in a fragment. So to understand what happened when rotating or leaving the application, you have to understand the lifecycle of fragment. 

The lifecycle of a fragment is very similar to the lifecycle of an activity. However, as the fragment is always attached to an activity, the first callback function of a fragment being called is onAttach, when the activity is just created. Following is the effect of the activity lifecycle on the fragment lifecycle presented by [google](http://developer.android.com/guide/components/fragments.html)

![placeholder](/images/graphview/activity_fragment_lifecycle.png)


#### Instantiate Fragment and setArgument

Google encourage the following pattern for the fragment instantiation and argument setting

- Use a static method and a no-argument construction to instantiate the fragment. Call setArgument(bundle) to set the member variables. I think this is to simplify the member variable's setting and retrieval( you don't have to write your own set/get functions)

{%  highlight java linenos  %}

    public static GraphFragment getInstance(int count) {
        if (frag == null) {
			frag = new GraphFragment();
			Bundle args = new Bundle();
			Graph graph = new Graph(count);
			args.putSerializable(ARG_GRAPH, graph);
			frag.setArguments(args);
        }
        return frag;
    }

    public GraphFragment() {
        setRetainInstance(false);
    }
	

{%  endhighlight  %}

- get arguments through getArgument

{%  highlight java linenos  %}

	@Override
	public void onCreate() {
		mGraph = (Graph)getArguments().getSerializable(ARG_GRAPH);
	}

{%  endhighlight  %}

### Handle configuration change

When the device is rotated, the configuration change event is triggered and the activity along with its fragments are recreated. However, if setRetainInstance(true) is called before, the fragment will not be destroyed at the moment. But onCreateView will be called. Therefore, we need to save the view state before rotation. I save the view state in onPause callback function through function setArgument. Here's the code.

{%  highlight java linenos  %}

    @Override
    public void onPause() {
        super.onPause();
        Log.d("GragphFragment", "onPause is called");
        Bundle args = getArguments();
        saveViewState(args);
    }

    private void saveViewState(Bundle args) {
        GraphView view = (GraphView)getView().findViewById(R.id.graphview);
        args.putIntArray(ARG_SORTEDNODE, view.getSortedNode());
        args.putParcelableArray(ARG_NODELIST, view.getNodeList());
        args.putSerializable(ARG_GRAPH, view.getGraph());
    }

{%  endhighlight  %}

View state is retrieved and applied when create view.

{% highlight java linenos %}
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View rootView =inflater.inflate(R.layout.graph_view, container, false);
        GraphView view = (GraphView)rootView.findViewById(R.id.graphview);
        Bundle args;
        if (savedInstanceState != null) {
            //try get args from savedInstanceState first, this is when fragement is recreated
            args = savedInstanceState;
        } else {
            //otherwise get args from bundle.
            args = getArguments();
        }

        if (args != null) {
            view.setGraph((Graph) args.getSerializable(ARG_GRAPH));
            if (!args.containsKey(ARG_NODELIST) || !args.containsKey(ARG_SORTEDNODE)) {
                view.reset();
            } else {
                Parcelable[] nodeList = args.getParcelableArray(ARG_NODELIST);
                //don't cast Parcelable[] to Rect[] directly. This is for type safety.
                view.setNodeList(toTypedList(nodeList).toArray(new Rect[nodeList.length]));
                view.setSortedNode(args.getIntArray(ARG_SORTEDNODE));
            }
         } else {
            view.setGraph(new Graph(Graph.DEFAULT_COUNT));
        }
        return rootView;
    }

    private <T> List<T> toTypedList(Parcelable[] ori) {
        List<T> dest = new ArrayList<T>();
        for (Parcelable item : ori) {
            dest.add((T)item);
        }
        return dest;
    }

{% endhighlight  %}

#### View state saving/retrieving
Sometimes fragment would be killed, for example, when the system is short of the memory and the activity is not active. In this situation, we need to save the view state. 

We save the state to activity's bundle in callback function onSaveInstanceState(). This function is called after calling onPause() and before onStop() is called. Here is the code.

{%  highlight java linenos  %}

    @Override
    public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        Log.d("GraphFragment", "onSaveInstanceState is called");
        saveViewState(outState);
    }

{%  endhighlight    %}

After doing above treatment, we no long need to worry about the state loss.

## Credit
- Cyril Mottier's [Deep Dive Into Android State Restoration](https://speakerdeck.com/cyrilmottier/deep-dive-into-android-state-restoration)
- [Saving (and Retrieving) Android Instance State](http://www.intertech.com/Blog/saving-and-retrieving-android-instance-state-part-2/)
- [Best practice for instantiating a new Android Fragment](http://stackoverflow.com/questions/9245408/best-practice-for-instantiating-a-new-android-fragment)
