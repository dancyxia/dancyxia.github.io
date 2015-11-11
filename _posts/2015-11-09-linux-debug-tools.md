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


For example, we want to find out if this code with regards to the json manipulation has any memory leak issue. 

{% highlight c linenos %}


#include <glib.h>
#include <stdlib.h>
#include <stdio.h>
#include "json_util.h"

int main()
{
        struct json_object *js;
        js = json_object_new_object();
        json_object_object_add(js, "key1", json_object_new_string( "test string.................1"));
        json_object_object_add(js, "key2", json_object_new_string( "test string.................2"));
        json_object_object_add(js, "key3", json_object_new_string( "test string.................3"));
        json_object_object_add(js, "key4", json_object_new_string( "test string.................4"));
        json_object_to_file("./test_json.txt", js); 

//        json_object_put(js);

}
{% endhighlight %}

{% highlight %}

Leak check _main_ detected leaks of 996 bytes in 16 objects
The 16 largest leaks:
Using local file ./test_json.
Leak of 512 bytes in 1 objects allocated from:
	@ 403387 lh_table_new
	@ 403439 lh_kchar_table_new
	@ 402719 json_object_new_object
	@ 40209e main
	@ 7f5fb0178ead __libc_start_main
	@ 401fa9 _start
Leak of 88 bytes in 1 objects allocated from:
	@ 40333b lh_table_new
	@ 403439 lh_kchar_table_new
	@ 402719 json_object_new_object
	@ 40209e main
	@ 7f5fb0178ead __libc_start_main
	@ 401fa9 _start

        ...
        ...
{%  endhighlight %}

Following is the output after running pprof. 

Using local file ./test_json.
Using local file /tmp/test_json.30247._main_-end.heap.
Total: 2 objects
       1  50.0%  50.0%        1  50.0% g_malloc ??:0
       1  50.0% 100.0%        1  50.0% json_escape_str /memory/json/json_object.c:101
       0   0.0% 100.0%        2 100.0% __libc_start_main /build/eglibc-ZhUScE/eglibc-2.13/csu/libc-start.c:244
       0   0.0% 100.0%        1  50.0% g_strdup ??:0
       0   0.0% 100.0%        1  50.0% json_escape_str /memory/json/json_object.c:98
       0   0.0% 100.0%        2 100.0% register_tm_clones crtstuff.c:0

### Use Valgrind
Valgrind is another very useful tool. Not like tcmalloc, you don't need to include any library. Valgrind just make your application run within its own run env. So all you need is just to install Valgrind. 

To make it detect the leak, just run this:

valgrind --leak-check=full ./test_json

And this is the output


==32724== Memcheck, a memory error detector
==32724== Copyright (C) 2002-2011, and GNU GPL'd, by Julian Seward et al.
==32724== Using Valgrind-3.7.0 and LibVEX; rerun with -h for copyright info
==32724== Command: ./test_json
==32724== 
==32724== 
==32724== HEAP SUMMARY:
==32724==     in use at exit: 1,252 bytes in 17 blocks
==32724==   total heap usage: 20 allocs, 3 frees, 1,476 bytes allocated
==32724== 
==32724== 1,252 (48 direct, 1,204 indirect) bytes in 1 blocks are definitely lost in loss record 17 of 17
==32724==    at 0x4C272B8: calloc (vg_replace_malloc.c:566)
==32724==    by 0x402421: json_object_new (json_object.c:161)
==32724==    by 0x4026BA: json_object_new_object (json_object.c:247)
==32724==    by 0x40207D: main (test_json.c:10)
==32724== 
==32724== LEAK SUMMARY:
==32724==    definitely lost: 48 bytes in 1 blocks
==32724==    indirectly lost: 1,204 bytes in 16 blocks
==32724==      possibly lost: 0 bytes in 0 blocks
==32724==    still reachable: 0 bytes in 0 blocks
==32724==         suppressed: 0 bytes in 0 blocks
==32724== 
==32724== For counts of detected and suppressed errors, rerun with: -v
==32724== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 4 from 4)

