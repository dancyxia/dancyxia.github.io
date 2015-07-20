---
layout: post
title: Codility Training Exercise 8 - Prime and composite numbers
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 8. 

**Done**: Exercise 1 - 3

**TO_BE_DONE**: Exercise 4

### Exercise 1: [CountFactors](https://codility.com/demo/take-sample-test/count_factors/) (Easy)
Count factors of given number n.
{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
	if (N == 1)
		return 1;
	//initialize count as 2, as for any number bigger than 1 at least have two factors: 1 and itself
        int count = 2;
        long i = 2;
        while (i * i < N) {
	    if (N%i == 0)
		count += 2;
	    i++;
        }
        if (i*i == N)
            count++;
        return count;
     }
}
{% endhighlight %}

### Exercise 2: [MinPerimeterRectangle](https://codility.com/demo/take-sample-test/min_perimeter_rectangle/) (Easy)
Find the minimal perimeter of any rectangle whose area equals N.

**Solution**:

{%  highlight java linenos  %}
class MinPerimSolution {
    public int solution(int N) {
        int minPerim= (1+N)<<1;
        int i = 2;
        while ( i*i < N)  {
            if (N%i==0) {
                minPerim = Math.min((i+N/i)<<1, minPerim);
            }
            i++;
        }
        return minPerim;
    }

    public static void main(String[] args) {
		//int[] A={3, 2, 6, -1, 4, 5, -1, 2};
		System.out.println(new MinPerimSolution().solution(36));
    }

}
{% endhighlight %}

### Exercise 3: [Peaks](https://codility.com/demo/take-sample-test/peaks/) (Moderate)
Divide an array into the maximum number of same((-))sized blocks, each of which should contain an index P such that A[P - 1] < A[P] > A[P + 1].

**Solution**:

Firstly get the peak sum total for every position. Then started from the small one, check every dividor to see if each block divided by the dividor contains at least one peak. 

{%  highlight java linenos  %}
class PeakSolution {
    public int solution(int[] A) {
        if (A.length < 3)
            return 0;

        int N = A.length;
        int[] peakSum = new int[N];
        for (int i = 1; i < N-1; i++) {
            peakSum[i] = peakSum[i-1];
            if (A[i] > A[i-1] && A[i] > A[i+1])
                peakSum[i]++;
        }
        peakSum[N-1] = peakSum[N-2];
        int i = 2;
        int minGroupNum = 0;
        while(i*i < N) {
            if (N%i == 0) {
                boolean ret = checkGroup(i, N, peakSum);
                if (ret)
                    return N/i;
                ret = checkGroup(N/i, N, peakSum);
                if (ret)
                    minGroupNum = i;
            }
            i++;
        }

        if (i*i == N && checkGroup(i, N, peakSum)) {
           return i;
        }

        if (minGroupNum == 0)
            return peakSum[N-1] >= 1 ? 1 : 0;
        return minGroupNum;
    }

    private boolean checkGroup(int num,int N, int[] peakSum) {

        int count = 1;
        boolean found = true;
        for (int j = num-1; j < N; j+=num) {
            if (peakSum[j] < count) {
                found = false;
                break;
            }
            count = peakSum[j]+1;
        }
        return found;
    }

    public static void main(String[] args) {
	int[] A={1, 2, 3, 4, 3, 4, 1, 2, 3, 4, 6, 2};
	System.out.printf("expected result: 3, my result: %d %n", new PeakSolution().solution(A));
    }
}

{% endhighlight %}

### Exercise 4: [Flags](https://codility.com/demo/take-sample-test/flags/) (Moderate)
Find the maximum number of flags that can be set on mountain peaks.

**Solution**:


{%  highlight java linenos  %}


{% endhighlight %}
