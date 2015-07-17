---
layout: post
title: Codility Training Exercise 4 - Sorting
category: job interview
tags: [algorithm]
---
This is the exercises for lesson 4. 

### Exercise 1: [Triangle](https://codility.com/demo/take-sample-test/triangle/)(Easy)

Determine whether a triangle can be built from a given set of edges.

**Solution**:

Sort it and try comparing the sum total of the minimum two number and the third minimum number. Be careful to the overflow possibility.

{%  highlight java linenos  %}
import java.util.Arrays;

class Solution {
    public int solution(int[] A) {
		if (A.length < 3) //PAY ATTENTION TO this
			return 0;
		Arrays.sort(A);
		for (int i = 2; i < A.length; i++) {
			
			if ((long)A[i-2]+(long)A[i-1] > A[i])
				return 1;
		}		
		return 0;
        
        // write your code in Java SE 8
    }
}
{% endhighlight %}

### Exercise 2: [MaxProductOfThree](https://codility.com/demo/take-sample-test/max_product_of_three/) (Easy) 
Maximize A[P] * A[Q] * A[R] for any triplet (P, Q, R).

**Solution**:

senario 1:
if there're more than two negative number and more than three positive number, the max product is either max1 * max2 * max3 or max1 * min1 * min2
senario 2:
if all are negative numbers, the max product is the one from the max three numbers
senario 3:
if there's only one or two positive number, the max product is from min1*min2*max
We can conclude that whatever the scenario is, the max product is either from max1*max2*max3 or max1*min1*min2. Therefore, we only need to keep the minimum two numbers and maximum three numbers. 

{%  highlight java linenos  %}
import java.util.Comparator;
import java.util.PriorityQueue;

class Solution {
    public int solution(int[] A) {
        PriorityQueue<Integer> minPQ = new PriorityQueue<>(4);
        PriorityQueue<Integer> maxPQ = new PriorityQueue<>(3, new Comparator<Integer>() {
            @Override
            public int compare(Integer integer, Integer integer2) {
                return integer2 - integer;
            }
        });

        for (int i = 0; i < A.length; i++) {
            maxPQ.add(A[i]);
            minPQ.add(A[i]);

            //only need to keep min 2 items and max 3 items
            if (maxPQ.size() ==3) maxPQ.poll();
            if (minPQ.size() ==4) minPQ.poll();
        }

        int temp1 = maxPQ.poll()*maxPQ.poll();
        int temp2 = minPQ.poll()*minPQ.poll();
        int max = minPQ.poll();
        return Math.max(temp1*max, temp2*max);
    }
}
{% endhighlight %}


### Exercise 3: [Distinct](https://codility.com/demo/take-sample-test/distinct/) (Easy)
Compute number of distinct values in an array. 

**Solution**:

It's straightforward. Use set to hold the items.

{%  highlight java linenos  %}
import java.util.TreeSet;

class DistinctSolution {
    public int solution(int[] A) {
		int N = A.length;
        
        if (N == 0) return 0;

        TreeSet<Integer> set = new TreeSet<>();
        for (int i = 0; i < N; i++)
            set.add(A[i]);
        return set.size();
    }
    
}
{% endhighlight  %}

### Exercise 4: [NumberOfDiscIntersections](https://codility.com/demo/take-sample-test/number_of_disc_intersections/)(Moderate))
Compute the number of intersections in a sequence of discs. 

**Solution**:

It's nothing about the shape. Sort (or put all to the priority queue) all the position of the left side and right side of the disc. And then scan each position. When meet a left side position, push the disc No to a set. If it's the right side position, remove the disc from the set. So when a left side position is scanned, the number of discs in the set is the number of intersections, in other words, all the discs in the set intersects with the coming disc.

{%  highlight java linenos  %}
import java.util.Comparator;
import java.util.HashSet;
import java.util.PriorityQueue;
import java.util.Set;

class Solution {
    enum PosType {LEFT, RIGHT};
    private static class Node {
        int pos;
        Integer discNo;
        PosType type;

        Node(int pos, int discNo, PosType type) {
            this.pos = pos;
            this.discNo = discNo;
            this.type = type;
        }
    }

    public int solution(int[] A) {
        if (A.length <= 1)
            return 0;

        PriorityQueue<Node> pq = new PriorityQueue<>(A.length<<1, new Comparator<Node>() {
            @Override
            public int compare(Node node, Node node2) {
                int res = node.pos - node2.pos;
                if (res == 0 && node.type != node2.type) {
                    return node.type == PosType.LEFT ? -1 : 1;
                }
                return res;
            }
        });

       for (int i = 0; i < A.length; i++) {
           pq.add(new Node(i-A[i], i, PosType.LEFT));
           pq.add(new Node(i+A[i], i, PosType.RIGHT));
       }

        Set<Integer> pool = new HashSet<>();
        int ret = 0;
        while(!pq.isEmpty()) {
            Node node = pq.poll();
            if (node.type == PosType.LEFT) {
                ret += pool.size();
                pool.add(node.discNo);
            } else
                pool.remove(node.discNo);
        }

        return ret;
    }

    public static void main(String[] args) {
		int[] A={1, 5, 2, 1, 4, 0};
		System.out.println(new Solution().solution(A));
    }
    
}

{% endhighlight %}
