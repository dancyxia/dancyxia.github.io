---
layout: post
title: Codility Training Exercise 11 - Fibonacci numbers
category: job interview
tags: [algorithm]
---
This is the exercises for lesson 11. 

**Done**: Exercise 1 

**Partly-Done**: Exercise 2. The solution has performance issue. Score 75%. 

### Exercise 1: [Ladder](https://codility.com/demo/take-sample-test/ladder/) (Easy)
Count the number of different ways of climbing to the top of a ladder.

**Solution**:

{%  highlight java linenos  %}
class LadderSolution {
    public int[] solution(int[] A, int[] B) {
		int N = A.length;
		if (N == 1)
			return new int[]{1};
        int[] check = new int[N+1];
        check[0] = 1;
        check[1] = 1;
        for (int i = 2; i<=N; i++)
			check[i] = check[i-1]+check[i-2];
        int[] ret = new int[A.length];
        for (int i = 0; i < A.length; i++) {
            int res = check[A[i]];
            ret[i] = res & mask(B[i]);
        }
        return ret;
    }

    private int mask(int num) {
        int ret = 1;
        for (int i = 0; i < num; i++) {
            ret = ret << 1;
        }
        return ret-1;
    }

    public static void main(String[] args) {
		int[] p={4, 4, 5, 5, 1};
		int[] q = {3, 2, 4, 3, 1};
		int[] ret = new LadderSolution().solution(p, q);
		for (int i = 0; i < ret.length; i++)
			System.out.print(ret[i]+" ");
    }
}
{% endhighlight %}

And here's C++ implementation
{%  highlight C++ linenos  %}
#include <iostream>
using namespace std;

int gcd(int N, int M) 
{
	if (N%M == 0)
		return M;
	return gcd(M, N%M);
}

int gcd2(int N, int M)
{
	if (N-M == 0)
		return M;
	if (N>M)
		return gcd2(N-M, M);
	else
		return gcd2(N, M-N);
}

int binGcd(int N, int M) {
	if (N==M) 					return N;
	else if ((N&1)==0 && (M&1)==0)	return binGcd(N>>1, M>>1)<<1;
	else if ((N&1)==0) 				return binGcd(N>>1, M);
	else if ((M&1)== 0) 				return binGcd(M>>1, N);
	else if (N>M)					return binGcd((N-M)>>1, M);
	else 							return binGcd((M-N)>>1, N);
}

int solution(int N, int M) {
	return N/gcd(N, M);
}

int main() 
{
	cout << solution(10, 4) << endl;
	return 0;
}
{% endhighlight %}

### Exercise 2: [FibFrog](https://codility.com/demo/take-sample-test/fib_frog/) (Moderate)

Count the minimum number of jumps required for a frog to get to the other side of a river.

**Solution**:

{%  highlight java linenos  %}
class LeapCountSolution {
    public int solution(int[] A) {
        int[] F = new int[A.length+3];
        F[0] = 1;
        F[1] = 1;
        for (int i=2; i < F.length; i++) {
            F[i] = F[i-1] + F[i-2];
        }

        int[] leapCount = new int[A.length+1];
        for (int i = 0; i < leapCount.length; i++)
            leapCount[i] = -1;
        return getLeapCount(leapCount.length-1, leapCount, F, A);
    }

    private int getLeapCount(int pos, int[] leapCount, int[] F, int[] A) {
        for (int i = 0; i < F.length; i++) {
            int lastPos = pos - F[i];
            if (lastPos == -1)
                return 1;
            if (lastPos < 0)
                break;
            if (A[lastPos] == 0)
                continue;
            if (leapCount[lastPos] == -1)
                leapCount[lastPos] = getLeapCount(lastPos, leapCount, F, A);
            if (leapCount[lastPos] == -1)
                continue;
            if (leapCount[pos] == -1)
                leapCount[pos] = Integer.MAX_VALUE;
            leapCount[pos] = Math.min(leapCount[pos], leapCount[lastPos] + 1);
        }

        return leapCount[pos];
    }

    public static void main(String[] args) {
//		int[] p={4, 4, 5, 5, 1};
//		int[] q = {3, 2, 4, 3, 1};
        int[] p = {0,0,0,1, 1,0,1,0,0,0,0};
		System.out.println(new LeapCountSolution().solution(p));
    }
}

{% endhighlight %}


