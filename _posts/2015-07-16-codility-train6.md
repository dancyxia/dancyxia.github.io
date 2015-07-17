---
layout: post
title: Codility Training Exercise 6 - Leader
category: job interview
tags: [algorithm]
---
This is the exercises for lesson 6. There're two exercises and I only did one. 

### Exercise 1: [Dominator](https://codility.com/demo/take-sample-test/dominator/) (Easy)
Find an index of an array such that its value occurs at more than half of indices in the array. 
{%  highlight java linenos  %}
{% endhighlight %}

### Exercise 2: [EquiLeader](https://codility.com/demo/take-sample-test/equi_leader/) (Easy)
Find the index S such that the leaders of the sequences A[0], A[1], ..., A[S] and A[S + 1], A[S + 2], ..., A[N - 1] are the same. 

**Solution**:

{%  highlight java linenos  %}
// you can also use imports, for example:
// import java.util.*;

// you can use System.out.println for debugging purposes, e.g.
// System.out.println("this is a debug message");

class EquiLeaderSolution {
    public int solution(int[] A) {
        if (A.length == 1)
            return 0;

        int toCompare = A[0];
        int count = 0;
        int value = -1;
        for (int i = 0; i<A.length; i++) {
            if (count == 0) {
                count++;
                value = A[i];
            } else if (value != A[i]) {
                count--;
            } else {
                count++;
            }
        }

        if (count > 0) {
            int sum = 0;
            for (int i = 0; i < A.length; i++) {
                if (A[i] == value) {
                    sum++;
                }
            }
            count = 0;
            int ret = 0;
            for (int i = 0; i < A.length; i++) {
                if (A[i] == value) {
                    count++;
                }
                if (count > ((i+1)>>1) && (sum-count) > ((A.length-i-1)>>1))
                    ret++;
            }
            return ret;
        }
        return 0;
    }
    // write your code in Java SE 8
}
{% endhighlight %}


