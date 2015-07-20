---
layout: post
title: Codility Training Exercise 9 - Sieve of Eratosthenes
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 9. 

**Done**: Exercise 1 - 2

### Exercise 1: [CountSemiprimes](https://codility.com/demo/take-sample-test/count_semiprimes/) (Easy)
Count the semiprime numbers in the given range [a..b]
{%  highlight java linenos  %}
class SemiPrimeCountSolution {
    public int[] solution(int N, int[] P, int[] Q) {
        int[] F = new int[N+1];
        for (int i = 1; i < F.length; i++) {
            F[i] = 0;
        }
        int i = 2;
        while (i*i <= N) {
            if (F[i] == 0) {
                int k = i * i;
                while (k <= N) {
                    if (F[k] == 0)
                        F[k] = i;
                    k+=i;
                }
            }
            i++;
        }
        
        int[] semiPrimSum = new int[N+1];
        for (int j = 1; j <= N; j++) {
		semiPrimSum[j] = semiPrimSum[j-1];
		if (isSemiPrim(j, F)) {
			semiPrimSum[j]+=1;
		}
	}

        int[] ret = new int[P.length];
        for (int j = 0; j < P.length; j++) {
		ret[j] = semiPrimSum[Q[j]] - semiPrimSum[P[j]-1];
        }

        return ret;
    }

    private boolean isSemiPrim(int num, int[] F) {
        if (F[num]==0) return false;
        return F[num/F[num]] == 0;
    }

    public static void main(String[] args) {
		int[] P={1, 4, 16};
		int[] Q={26, 10, 20};
		int[] ret = new SemiPrimeCountSolution().solution(26, P, Q);
		for (int t : ret) {
			System.out.print(t+" ");
		}
    }
{% endhighlight %}

### Exercise 2: [CountNonDivisible](https://codility.com/demo/take-sample-test/count_non_divisible/) (Moderate)
Calculate the number of elements of an array that are not divisors of each element.

**Solution**:

{%  highlight java linenos  %}
import java.util.HashSet;
import java.util.Set;

class CountNonDivisibleSolution {
    public int[] solution(int[] A) {
        int N = A.length<<1;
        int[] mark = new int[N+1];
        for (int i = 0; i < A.length; i++) {
            mark[A[i]]++;
        }

        int[] divisorCount = new int[N+1];
        for (int i = 0; i < A.length; i++) {
            if (mark[A[i]] > 0) {
                int k = A[i];
                while (k <= N) {
                    divisorCount[k] += mark[A[i]];
                    k += A[i];
                }
                mark[A[i]] = 0;
            }
        }

        int[] ret = new int[A.length];
        for (int i = 0; i < ret.length; i++) {
            ret[i] = A.length - divisorCount[A[i]];
        }

        return ret;
    }


    public static void main(String[] args) {
		int[] P={3, 1, 2, 3, 6};
		int[] ret = new CountNonDivisibleSolution().solution(P);
		for (int t : ret) {
			System.out.print(t+" ");
		}
		
    }
}
{% endhighlight %}


