---
layout: post
title: Cusom Annotation
category: job-interview
tags: [algorithm, annotation]
---

### Custom Annotation

I got a job interview notice. Now I have to suspend my adventure on GraphDemo application and prepare for the technical exercise. The due date of the exercise is next Friday. So I have enough time to prepare for it. I plan to finish the reading of "cracking the coding interview" and write all the exercises from the book. As the first step, I write a simple test framework for the exercise practising. 

The framework is based on java annotation. Similart to Junit test, I add the custom annotation to every test function. And then these functions can be executed one by one in the main function.

Generally, setting up a custom annotation would go through following steps.

#### Define annotation interface

{%  highlight java linenos  %}

@Retention(RetentionPolicy.RUNTIME)
@Target (ElementType.METHOD)
public @interface TestExercise {
	public boolean enabled() default true;
}

{% endhighlight %}

Annotation @Retention and @Target are built in annotations and served as the meta annotations.
- @Retention
This annotation specifies the functional scope of the custom annotation. It has following possible values:
  - SOURCE: this indicates that this annotation is only retained in resource. It's ignored by compiler and JVM
  - CLASS: if this value is presented, the annotation is retained by the compiler, but will be ignored by JVM
  - RUNTIME: this indicates that this annotation will be retained in JVM and can be used at runtime via reflection.
Here I use RUNTIME.  

- Target
This annotation indicates the type of the custom annotation. Possible values can be: ANNOTATION_TYPE, CONSTRUCTOR, FIELD, TYPE, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER. 

I use METHOD for our test annotation

I also declared a property: enabled. This is to indicate if a test method should be executed.

#### Retrieve annotation

{%  highlight java linenos %}
	public static void main (String args[]) {
		try {
			Class testContainer = Class.forName("TestContainer");
			Class testEntity = Class.forName("test.TestExercise");
			for (Method m : testContainer.getDeclaredMethods()) {
				if (m.isAnnotationPresent(testEntity)) {
					TestExercise test = (TestExercise)m.getAnnotation(testEntity);
					if (test.enabled()) {
						System.out.printf("Test method: %s %n", m.getName());
						m.invoke(testContainer.newInstance());
						System.out.println();
					}
				}
			}
		} catch(Exception e) {
			e.printStackTrace();
		}
	}
{% endhighlight  %}
Several java reflection methods are used to retrieve annotation. function getDeclaredMethods is used to retrieve all declared methods of the class. Then isAnnotationPresent function is used to check if the custom annotation is presented. Function getAnnotation is for getting the annotation. And then we can use the annotation to check property "enabled".  

#### Place annotation
Finally have a look at how the custom annotation is placed for the test function to be executed.
{% highlight java linenos %}
	@TestExercise(enabled = true)
	public void testCompress() {
		String s = "aaaaab eeererrrrrrrre ddffffdf";
		char[] ori = s.toCharArray();
		char[] input = new char[ori.length+10];
		System.arraycopy(ori, 0, input, 0, ori.length);
     	System.out.printf("%s is compressed to %s %n", s, StringExercise.compress(s));
	}
{% endhighlight  %}

### test result

![placeholder](/images/jobinterview/test-result.png)

## Credit
- [Java Annotations Tutorial – The ULTIMATE Guide](http://www.javacodegeeks.com/2014/11/java-annotations-tutorial.html)
