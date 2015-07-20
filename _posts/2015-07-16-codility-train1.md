---
layout: post
title: Codility Training Exercise 1 - Time Complexity
category: job-interview
tags: [algorithm]
---

I went for three coding interview. Succeeded in one and failed in two. The failure was because I was not fast enough to finish the coding. I have another coding interview to go. To make myself to be more efficient, I tried to finish the exercises from codility as much as possible before the next interview. Hope it helped landing the job!

**Done**: Exercise 1 - 3

### Exercise 1: [TapeEquilibrium](https://codility.com/demo/take-sample-test/tape_equilibrium/) (Easy) 
 Minimize the value |(A[0] + ... + A[P-1]) - (A[P] + ... + A[N-1])|. 

This is straightforward. Be careful about the corner case.


{%  highlight java linenos  %}
class Solution {
    public int solution(int[] A) {
        int N = A.length;

        int[] sum = new int[N+1];
        for (int i = 1; i <= N; i++) {
            sum[i] = sum[i-1]+A[i-1];
        }

	int minDiff = Math.abs((sum[1]<<1)-sum[N]);
        for (int i = 2; i < N; i++) { 
	  minDiff = Math.min(minDiff, Math.abs((sum[i]<<1) - sum[N]));
        }

        return minDiff;
    }

    public static void main(String[] args) {
	int[] A={3, 1, 2, 4, 3}; 
	System.out.println(new Solution().solution(A));
    }
}

{% endhighlight %}

### Exercise 2: [FrogJmp](https://codility.com/demo/take-sample-test/frog_jmp/) (Easy) 
Count minimal number of jumps from position X to Y. 

{%  highlight java linenos  %}
class Solution {
    public int solution(int X, int Y, int D) {
        // write your code in Java SE 8
        int distance = Y-X;
        return distance%D == 0? distance/D : distance/D+1;
    }
}
{% endhighlight %}


### Exercise 3: [PermMissingElem](https://codility.com/demo/take-sample-test/perm_missing_elem/) (Easy) 
Find the missing element in a given permutation

**Solution**:

Simply add up the difference between every index+1 and the array value. The sum result would be the missing value - (N+1). Set the initial sum value as N+1, then the sum up value is the missing value.  

{% highlight java linenos %}
class Solution {
    public int solution(int[] A) {
		int N = A.length;
		long sum = N+1;
		for (int i = 0; i < N; i++)
			sum +=i+1-A[i];
		return (int)sum;
    }

    public static void main(String[] args) {
		int[] A={3, 1, 2, 6, 5}; 
		System.out.println(new Solution().solution(A));
    }

}
{% endhighlight  %}

Similar to above solution, you can also use XOR method to get the missing number.

Another different solution is by swapping the numbers to place every element in the right place. The place where hold the "N+1" element is supposed to hold the missing number.

{% highlight java linenos %}
class Solution {
    public int solution(int[] A) {
		int N = A.length;
		int holderMiss = N; //if there's no missing element, return N+1
		for (int i = 0; i < N; i++) {
			while (A[i] != i+1 && A[i] != N+1) {
				int t = A[A[i]-1];
				A[A[i]-1] = A[i];
				A[i] = t;
			}
			if (A[i] == N+1)
				holderMiss = i;
		}
		
		return holderMiss+1;
	}
}
{% endhighlight  %}
