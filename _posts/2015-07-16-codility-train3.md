---
layout: post
title: Codility Training Exercise 3 - Prefix Sums
category: job interview
tags: [algorithm]
---
This is the exercises for the third lesson. I finished part of it.

### Exercise 1: CountDiv (Easy)[TO BE FINISHED] 

Compute number of integers divisible by k in range [a..b]. 

[detail description](https://codility.com/demo/take-sample-test/count_div/)

{%  highlight java linenos  %}
{% endhighlight %}

### Exercise 2: PassingCars (Easy) [TO BE FINISHED]
Count the number of passing cars on the road

[detail description](https://codility.com/demo/take-sample-test/passing_cars/)

{%  highlight java linenos  %}
{% endhighlight %}


### Exercise 3: MinAvgTwoSlice (Moderate)
Find the minimal average of any slice containing at least two elements. 

[detail description](https://codility.com/demo/take-sample-test/min_avg_two_slice/)

**Solution**:
Solution is based on this statement: The minimum average number must be in the slice with 2 or 3 items. 
##Proof##:
1. every slice can be divided to sub slices which have only two or three items
2. the average of a slice must be equal or bigger than one of its sub slice. It's true for the the equal assumption. If the average of the subslices are different, then there must be some bigger sub slices and some other smaller ones. 
{% highlight java linenos %}
//3.3 MinAvgTwoSlice 
//Find the minimal average of any slice containing at least two elements
class MinAvgSliceSolution {
    public int solution(int[] A) {
        int N = A.length;
        if (N < 3)
			return 0;
        //first pass to get the sum total between A[0] to A[i]. Be noted that the length of sum array is N+1
        int[] sum = new int[N+1];
        sum[1] = A[0];
        for (int i = 2; i <= N; i++) {
            sum[i] = sum[i-1]+A[i-1];
        }

		//second pass: get slice with smallest average count
        float minAvg = Math.min((float) (sum[2] - sum[0]) / 2, (float) (sum[3] - sum[0]) / 3);
        int minStart = 0;
        for (int i = 3; i <= N; i++) {
            float avg = (float)(sum[i]-sum[i-2])/2;
            if (i < N) {
                avg = Math.min(avg, (float) (sum[i + 1] - sum[i - 2]) / 3);
            }
            if (avg < minAvg)
            {
                minAvg = avg;
                minStart = i-2;
            }
        }

        return minStart;
    }

    public static void main(String[] args) {
		int[] A={4, 2, 2, 5, 1, 5, 8};
		System.out.println(new MinAvgSliceSolution().solution(A));
    }
    
}

{% endhighlight  %}

### Exercise 4: MaxCounters  (Moderate))[TO BE FINISHED]
Calculate the values of counters after applying all alternating operations: increase counter by 1; set value of all counters to current maximum. 

[detail description](https://codility.com/demo/take-sample-test/max_counters/)
{%  highlight java linenos  %}
{% endhighlight %}
