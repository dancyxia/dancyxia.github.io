---
layout: post
title: Single thread TCP server by using linux's epoll facility
category: linux-network
tags: [linux epoll socket]
---

### TCP server

As a TCP server, it implements at least following functions:
1. It can accept connections from multiple clients
2. It can handle data passed from clients and return corresponding data to the clients.

To set up connections and do the communication, socket goes to the play. In network programming, a socket is like a file handler, just instead of representing a file, it represents a pairing endpoint in the network. When writing or reading from a socket, the data are passed to or from this paring endpoint. 
One server can have thousands of connections. How to manage such huge number of connections? Have a thread or process for each connection? It's not an ideal solution as effort to manage shared resources is challenging. Linux offered a notification facility called epoll which enables to manage the multiple connections in one thread.

### socket
For a TCP server, socket set consists of functions socket(), bind() and accept(). See the following code for how to use them

```c
int setup_server_socket(char *port)
{
    struct addrinfo hints;
    struct addrinfo *res, *ori_res;
    hints.ai_family = AF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_PASSIVE;    /* For wildcard IP address */
    hints.ai_protocol = SOL_TCP;          /* Any protocol */
    hints.ai_canonname = NULL;
    hints.ai_addr = NULL;
    hints.ai_next = NULL;
    int result = getaddrinfo(NULL, port, &hints, &ori_res);
    if (result != 0) {
        fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(result));
        exit(EXIT_FAILURE);
    }

    int fd=-1;
    int on=1;
    for (res=ori_res; res; res=res->ai_next) {
        fd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
        if (fd == -1) continue;
        if (setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on)) == -1) {
            fprintf(stderr, "set SO_REUSEADDR failed: %m");
            goto error_out;
        }

        //TCP_DEFER_ACCEPT (since Linux 2.4)
        //        Allow a listener to be awakened only when data arrives on the
        //        socket.  Takes an integer value (seconds), this can bound the
        //        maximum number of attempts TCP will make to complete the
        //        connection.  This option should not be used in code intended
        //        to be portable.
        //http://unix.stackexchange.com/questions/94104/real-world-use-of-tcp-defer-accept/94120#94120
        if (setsockopt(fd, IPPROTO_TCP, TCP_DEFER_ACCEPT, &on, sizeof(on)) == -1) {
            fprintf(stderr, "set TCP_DEFER_ACCEPT failed: %m");
            goto error_out;
        }
        if (!make_non_block_socket(fd)) {
            goto error_out;
        }
        if (fcntl(fd, F_SETFD, FD_CLOEXEC) == -1) {
            fprintf(stderr, "set CLOSE_ON_EXEC flag failed: %m");
            goto error_out;
        }
        if (bind(fd, res->ai_addr, res->ai_addrlen) == 0) {
            if (listen(fd, BACKLOG) == -1) {
                fprintf(stderr, "listen failed \n");
                goto error_out;
            }
            break;
        }

error_out:      
		close(fd);
        fd = -1;
    }
	
    if (fd == -1) {
        fprintf(stderr, "socket is not established!\n");
        exit(EXIT_FAILURE);
    }

    if (ori_res)
        freeaddrinfo(ori_res);

    return fd;
}

void accept_connection(int socket, accept_connection_callback_t callback)
{
    struct sockaddr in_addr;
    socklen_t in_len = sizeof in_addr;
    int infd;
    char hbuf[NI_MAXHOST], sbuf[NI_MAXSERV];
    infd = accept(socket, &in_addr, &in_len);
    if (infd == -1) {
        if (errno != EAGAIN && errno != EWOULDBLOCK)
        {
            fprintf(stderr, "epoll accept failed");
        }
        return;
    }

    if (getnameinfo(&in_addr, in_len, hbuf, sizeof hbuf, sbuf, sizeof sbuf, NI_NUMERICHOST|NI_NUMERICSERV) == 0) {
        printf("Accepted service is on host %s, port %s \n", hbuf, sbuf);
    }

    //add fd to epoll monitor
    if (make_non_block_socket(infd)) {
        if (!callback(infd)) {
            goto error_out;
        }
    } else {
        goto error_out;
    }

    return;
error_out:
    close(infd);
}

```

### EPOLL 
epoll set consists of three system call. epoll_create, epoll_ctl and epoll_wait.

#### epoll_create and epoll_create1

```c
#include <sys/epoll.h>
int epoll_create(int size);int epoll_create1(int flags);
```

epoll_create() or epoll_create1() is to create and return a file descriptor referring to the created epoll instance. It serves as the epoll interface for subsequent epoll calls. epoll_create was added to the kernel in version 2.6. epoll_create1() was added to the kernel in version 2.6.27. The size parameter for epoll_create() is original designed as the initial number of the file descriptors to be added to the epoll instance. As the design change, now this value is no longer used. However, to be back compatible, the value needs to be greater than 0.

for epoll_create1(), argument flags can include EPOLL_CLOEXEC to set the close-on-exec (FD_CLOEXEC) flag on the new file descriptor. 

the fd created by epoll_create must be closed with close(fd) to release the allocated resources when it's no longer used.


#### epoll_ctl

```c
#include <sys/epoll.h>

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

funciton epoll_ctl() is used to control epoll operations. 

The first argument is the fd of the epoll instance.
The second argument specify the epoll operation. The valid operations are EPOLL_CTL_ADD, EPOLL_CTL_MOD and EPOLL_CTL_DEL. 
The third argument is the file descriptor to be worked on. The event argument describes the object linked to the file descriptor fd. 
The struct epoll_event is defined as 

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```

The events member is a bit set composition. The example only handle events EPOLLIN, EPOLLET and EPOLLRDHUP

#### epoll_wait

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *events,
               int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events,
               int maxevents, int timeout,
               const sigset_t *sigmask);
```
The function epoll_wait and epoll_pwait waits for events on the epoll(7) instance referred to by the file descriptor epfd. The memory events pointer point to stores the events information. The argument maxevents must be greater than 0. Argument timeout specifies the 
minimum number of milliseconds that epoll_wait() will block. specifying it as -1 cause the epoll_wait() to block infinitely, while 0 makes the epoll_wait() to return immediately. 

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