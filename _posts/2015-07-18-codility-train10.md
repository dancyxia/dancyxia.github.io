---
layout: post
title: Codility Training Exercise 10 - Euclidean algorithm
category: job-interview
tags: [algorithm]
---
This is the exercises for lesson 10. 

**Done**: Exercise 1 - 2

### Exercise 1: [ChocolatesByNumbers](https://codility.com/demo/take-sample-test/chocolates_by_numbers/) (Easy)
There are N chocolates in a circle. Count the number of chocolates you will eat. 

**Solution**:

If strenching the circle to a line, The place that the first chocolate is revisited(by proceed with step M) actually is the same one that proceeding with step N. Therefore the place is LCM(Least Common Multiple). Thus the number of chocolates you eat is LCM/M=(N*M/GCD)/M=N/GCD.

{%  highlight java linenos  %}
class Solution {
    public int solution(int N, int M) {
        return (int)N/gcd(N, M);
    }

    private int gcd(int N, int M) {
        if (N%M == 0)
            return M;
        return gcd(M, N%M);
    }
    

    public static void main(String[] args) {
	System.out.printf("expected number is 5, the result is %d %n", new Solution().solution(10, 4));
		
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

### Exercise 2: [CommonPrimeDivisors](https://codility.com/demo/take-sample-test/common_prime_divisors/) (Moderate)
Check whether two numbers have the same prime divisors.

**Solution**:

{%  highlight java linenos  %}
class CommonPrimeDivisorSolution {
    public int solution(int[] A, int[] B) {
        int count = 0;
        for (int i =0; i < A.length; i++) {
            int gcd1 = gcd(A[i], B[i]);
            int gcdt;
            int a = A[i]/gcd1;
            while((gcdt=gcd(gcd1, a))!=1) {
				a /= gcdt;
			}
            int b = B[i]/gcd1;
            while((gcdt=gcd(gcd1, b))!=1) {
				b /= gcdt;
			}
			if (a == 1 && b == 1)
				count++;
        }
        return count;
    }
    
    private int gcd(int a, int b) {
        if (a % b == 0) return b;
        return gcd(b, a%b);
    }

    public static void main(String[] args) {
		int[] p={15, 10, 3};
		int[] q = {75, 30, 5};
		System.out.println(new CommonPrimeDivisorSolution().solution(p, q));
    }
}

{% endhighlight %}


