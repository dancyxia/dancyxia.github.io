---
layout: post
title: Codility Training Exercise 7 - Slice Sum
category: job interview
tags: [algorithm]
---
This is the exercises for lesson 7. 

**Done**: Exercise 1 - 3

### Exercise 1: [MaxSliceSum](https://codility.com/demo/take-sample-test/max_slice_sum/) (Easy)
Find a maximum sum of a compact subsequence of array elements. 
{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
		int N = A.length;
        if (N == 1)
            return A[0];
        
        int maxSum = A[0];    
        int sum = A[0];
        for (int i = 1; i < A.length; i++) {
            if (sum < 0)
				sum = 0;
			sum += A[i];
			maxSum = Math.max(maxSum, sum);
		}

        return maxSum;
     }
}
{% endhighlight %}

### Exercise 2: [MaxProfit](https://codility.com/demo/take-sample-test/max_profit/) (Easy)
Given a log of stock prices compute the maximum possible earning. 

**Solution**:

{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
        int maxStart = 0;
        int maxProfix = 0;
        int start = 0;
        for (int i = 0; i < A.length; i++) {
            if (A[i] - A[start] > maxProfix) {
                maxProfix = A[i] - A[start];
                maxStart = start;
            }

            if (A[i] < A[maxStart])
                start = i;
        }

        return maxProfix;
    }
}
{% endhighlight %}

### Exercise 3: [MaxDoubleSliceSum](https://codility.com/demo/take-sample-test/max_double_slice_sum/) (Easy)
 Find the maximal sum of any double slice. 

**Solution**:

Do two scanning. The first pass is from 1 to N, the other pass is from N to 1. When doing the first pass, keep all sum total so that it can be used in second pass.

{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
        int maxStart = 0;
        int maxProfix = 0;
        int start = 0;
        for (int i = 0; i < A.length; i++) {
            if (A[i] - A[start] > maxProfix) {
                maxProfix = A[i] - A[start];
                maxStart = start;
            }

            if (A[i] < A[maxStart])
                start = i;
        }

        return maxProfix;
    }
}
{% endhighlight %}



