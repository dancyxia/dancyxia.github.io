---
layout: post
title: Codility Training Exercise 5 - Stack and Queue
category: job interview
tags: [algorithm]
---
This is the exercises for lesson 5. It's easy. I only did one exercise. 

### Exercise 1: [Nesting](https://codility.com/demo/take-sample-test/nesting/) (Easy)
Determine whether given string of parentheses is properly nested. 
{%  highlight java linenos  %}
{% endhighlight %}

### Exercise 2: [StoneWall](https://codility.com/demo/take-sample-test/stone_wall/) (Easy)
Cover "Manhattan skyline" using the minimum number of rectangles. 

**Solution**:

{%  highlight java linenos  %}
// you can also use imports, for example:
// import java.util.*;

// you can use System.out.println for debugging purposes, e.g.
// System.out.println("this is a debug message");

import java.util.Stack;
class StoneWallSolution {
    public int solution(int[] H) {
        int count = 0;
        Stack<Integer> stack = new Stack<>();
        stack.push(H[0]);
        count++;
        for (int i = 1; i < H.length; i++) {
            while (!stack.isEmpty() && H[i] < stack.peek() ) {
                stack.pop();
            }
            int newHeight;
            if (stack.isEmpty())
                newHeight = H[i];
            else
                newHeight = H[i] - stack.peek();
            if (newHeight > 0) {
                stack.push(H[i]);
                count++;
            }
        }
        return count;
    }

    public static void main(String[] args) {
		int[] A={8, 8, 5, 7, 9, 8, 7, 4, 8};
		System.out.println(new StoneWallSolution().solution(A));
    }
    
}

{% endhighlight %}


### Exercise 3: [Brackets](https://codility.com/demo/take-sample-test/brackets/) (Easy)
Determine whether a given string of parentheses is properly nested. 

**Solution**:

{%  highlight java linenos  %}
{% endhighlight  %}

### Exercise 4: [Fish]() (Easy)
N voracious fish are moving along a river. Calculate how many fish are alive. 

**Solution**:

{%  highlight java linenos  %}
{% endhighlight %}
