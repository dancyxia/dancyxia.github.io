---
layout: post
title: Codility Training Exercise 12 - Binary search algorithm
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 12. 

**Done**: Exercise 1 

**To-Be-Done**: Exercise 2. 

### Exercise 1: [MinMaxDivision](https://codility.com/demo/take-sample-test/min_max_division/) (Moderate)
Divide array A into K blocks and minimize the largest sum of any block.

**Solution**:

{%  highlight java linenos  %}
class MinMaxDivisionSolution {
    public int solution(int K, int M, int[] A) {
        if (M == 0)
            return 0;

        if (A.length == 1)
            return A[0];

        int lo = 0;
        int hi=0;
        for (int item : A) {
            lo = Math.max(item, lo);
            hi += item;
        }

        int ret = -1;
        while (lo <= hi) {
            int mid = (lo+hi)>>1;
            if (check(mid, K, A)) {
				ret = mid;
                hi = mid - 1;
			}
            else
                lo = mid + 1;
        }

        return ret;
    }

    private boolean check(int maxSum,int k, int[] A) {
        int sum = 0;
        for (int i = 0; i < A.length-1; i++) {
            sum += A[i];
            if (sum + A[i+1] > maxSum) {
                sum = 0;
                k--;
                if (k == 0)
                    return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
		int[] A={2, 1, 5, 1, 2, 2, 2};
		System.out.printf("Expected result: 6, actual result: %d %n", new MinMaxDivisionSolution().solution(3, 5, A));
    }
}
{% endhighlight %}

### Exercise 2: [NailingPlanks](https://codility.com/demo/take-sample-test/nailing_planks/) (Difficult)

Count the minimum number of nails that allow a series of planks to be nailed.


**Solution**:
TO-BE-DONE
{%  highlight java linenos  %}
{% endhighlight %}


