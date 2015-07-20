---
layout: post
title: Persist the configuration
category: android-app-graphgen
tags: [algorithm, android]
---

### Use shared preference

Almost every application has some configurable data, our Graph Demo has no exception. In android, usually we use shared preference to persist the configurable  data.

#### Setting entrance

First, we add a menu item in the option menu as the configuration entrance. 

{% highlight text linenos %}
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools" tools:context=".MainActivity">
    <item android:id="@+id/action_settings"
        android:title="@string/action_settings"
        android:icon="@drawable/ic_action"
        android:orderInCategory="100"
        app:showAsAction="ifRoom" />
</menu>

{% endhighlight %}

Please be noted that attribute showAsAction is in app scope other than android scope. This is because I imported the supporting library for low level(<API Level 11) android support. 

#### Create preference.xml 

We need to create a preference xml document to claim the preference information. This document should be placed at res/xml folder. Usually you have to create xml folder manually.

{% highlight text linenos %}
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <EditTextPreference
        android:key="@string/pref_graphnodecount"
        android:title="@string/graph_nodenum"
        android:inputType="number"
        android:defaultValue="5" />
</PreferenceScreen>
{% endhighlight %}

Preference items are placed inside PreferenceScreen element. Attribute key is required for every preference item. Android provides various type of preference. The most common preferences are: CheckBoxPreference, ListPreference and EditTextPreference. If you have too many preferences, categorize them can provide more friendly experience to the user. 

#### Create SettingFragment by extending PreferenceFragement and its containing activity
Handler for the preference can be either PreferenceActivity or PreferenceFragment. It's encouraged to use PreferenceFragment if the target device is Adroid 3.0+.

{% highlight java linenos %}
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        addPreferencesFromResource(R.xml.preferences);

        PreferenceScreen ps = getPreferenceScreen();
        for (int i = 0; i < ps.getPreferenceCount(); i++) {
            Preference p = ps.getPreference(i);
            initSummary(p);
        }

        Preference p = ps.findPreference(getResources().getString(R.string.pref_graphnodecount));
        if (p != null) {
            p.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
                @Override
                public boolean onPreferenceChange(Preference preference, Object newValue) {
                    String sVal = (String)newValue;
                    StringBuilder sb = new StringBuilder();
                    try {
                        int val = Integer.parseInt(sVal);
                        if (val < Graph.MINNODENUM || val > Graph.MAXNODENUM) {
                            sb.append("The value should be in the scope of ").append(Graph.MINNODENUM).append(" and ").append(Graph.MAXNODENUM);
                        }
                    } catch (NumberFormatException e) {
                        sb.append(e.getMessage()).append("\n");
                    }
                    if (sb.length() > 0) {
                        new AlertDialog.Builder(getActivity()).setTitle("Error").setMessage(sb.toString()).setPositiveButton(android.R.string.ok, null).show();
                        return false;
                    }
                    return true;
                }
            });
        }
    }
{% endhighlight %}

From above code snippet you can find the preference xml is linked through function addPreferencesFromResource(R.xml.preferences). This is different from what general activity or fragment does.

#### Preference retrieval
To retrieve the preference from SharedPreference, you can use following code.

{%  highlight java linenos %}
        int nodeSize = Integer.parseInt(PreferenceManager.getDefaultSharedPreferences(getApplicationContext()).getString(getResources().getString(R.string.pref_graphnodecount), String.valueOf(Graph.DEFAULT_COUNT)));
{% endhighlight %}

Note: the sharedpreference is under the application context. All activities with the same application have the access to it. 

#### Pass the value to preference setting
Then after the application is started and the value is loaded to SharedPreference. How is the change passing to the preference fragment? The answer is straightforward: have fragment implement SharedPreferences.OnSharedPreferenceChangeListener. Here is the code.

{% highlight java linenos %}
    @Override
    public void onResume() {
        super.onResume();
        getPreferenceScreen().getSharedPreferences().registerOnSharedPreferenceChangeListener(this);
    }

    @Override
    public void onPause() {
        super.onPause();
        getPreferenceScreen().getSharedPreferences().unregisterOnSharedPreferenceChangeListener(this);
    }

    @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key) {
        Preference p = findPreference(key);
        if (p instanceof EditTextPreference) {
            ((EditTextPreference)p).setSummary(((EditTextPreference) p).getText());
        }
    }
{%   endhighlight  %}


#### Validate the preference
You can also validate a specific preference before the value is passed to SharedPreference

{% highlight java linenos %}
    @Override
    public void onCreate(Bundle savedInstanceState) {

	     ...
		 ...
        Preference p = ps.findPreference(getResources().getString(R.string.pref_graphnodecount));
        if (p != null) {
            p.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
                @Override
                public boolean onPreferenceChange(Preference preference, Object newValue) {
                    String sVal = (String)newValue;
                    StringBuilder sb = new StringBuilder();
                    try {
                        int val = Integer.parseInt(sVal);
                        if (val < Graph.MINNODENUM || val > Graph.MAXNODENUM) {
                            sb.append("The value should be in the scope of ").append(Graph.MINNODENUM).append(" and ").append(Graph.MAXNODENUM);
                        }
                    } catch (NumberFormatException e) {
                        sb.append(e.getMessage()).append("\n");
                    }
                    if (sb.length() > 0) {
                        new AlertDialog.Builder(getActivity()).setTitle("Error").setMessage(sb.toString()).setPositiveButton(android.R.string.ok, null).show();
                        return false;
                    }
                    return true;
                }
            });
        }
		
		...
		...
    }

{% endhighlight %}

OK, now the configuration feature is ready. Next, we need to add feature to load/save the generated graph.