---
layout: post
title: Codility Training Exercise 15 - Dynamic programming
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 15.

**Done**: Exercise 1

**To-Be-Done**: Exercise 2

### Exercise 1: [NumberSolitaire](https://codility.com/demo/take-sample-test/number_solitaire/) (Moderate)
Find a maximal set of non((-))overlapping segments.

**Solution**:

{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
        int[] dp = new int[A.length];
        for (int i = 0; i < dp.length; i++)
            dp[i] = Integer.MIN_VALUE;
        
        dp[0] = A[0];
        for (int i = 0; i < A.length; i++) {
            for (int j = 1; j <= 6; j++) {
                if (i+j < A.length)
                    dp[i+j] = Math.max(dp[i+j], dp[i]+A[i+j]);
                else
                    break;
            }
        }
        return dp[A.length-1];
    }
}{% endhighlight %}

### Exercise 2: [MinAbsSum](https://codility.com/demo/take-sample-test/min_abs_sum/) (Difficult)

Given array of integers, find the lowest absolute sum of elements..


**Solution**:
TO-BE-DONE
{%  highlight java linenos  %}
{% endhighlight %}


