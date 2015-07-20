---
layout: post
title: Codility Training Exercise 14 - Greedy algorithms
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 14. I didn't do exercise 13. 

**Done**: Exercise 1 - 2

### Exercise 1: [MaxNonoverlappingSegments](https://codility.com/demo/take-sample-test/max_nonoverlapping_segments/) (Easy)
Find a maximal set of non((-))overlapping segments.

**Solution**:

{%  highlight java linenos  %}
{% endhighlight %}

### Exercise 2: [TieRopes](https://codility.com/demo/take-sample-test/tie_ropes/) (Easy)

Tie adjacent ropes to achieve the maximum number of ropes of length >= K.


**Solution**:

{%  highlight java linenos  %}
class TieRopesSolution {
    public int solution(int K, int[] A) {
        if (A.length == 1)
            return A[0] >= K? 1 :0;
        int count = 0;
        int sum = 0;
        for (int i = 0; i < A.length; i++) {
            if ((sum += A[i]) >= K) {
                sum = 0;
                count++;
            }
        }
        return count;
    }

    public static void main(String[] args) {
		//~ int[] p={1, 3, 7, 9, 9};
		//~ int[] q = {5, 6, 8, 9, 10};
		int[] p = {1, 2, 3, 4, 1, 1, 3};
		System.out.println(new TieRopesSolution().solution(4, p));
    }
}

{% endhighlight %}


