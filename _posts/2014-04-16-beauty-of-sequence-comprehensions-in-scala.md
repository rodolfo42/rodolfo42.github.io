---
published: true
layout: post
title: The beauty of sequence comprehensions in Scala
comments: true
---

While working through some [projecteuler.net](http://projecteuler.net/) problems, I've found a quite nice solution to a problem using Scala's sequence comprehensions.

The problem was to *compute the largest palindrome number that is a product of two numbers consisting of three algarisms each*.

<!-- more -->

Firstly, a palindrome number is one that reads both ways:

{% highlight scala %}
def pal(a: Int): Boolean =
	a.toString == a.toString.reverse
{% endhighlight %}

Then use a sequence comprehension to find all the palindrome products, filtering by the ´pal´ function:

{% highlight scala %}
def greatestPal(from: Int, until: Int): Int = {
	lazy val pals = for {
		x <- start to end
		y <- start to end
		if pal(x * y)
	} yield x * y

	pals.max
}
{% endhighlight %}

Reads nicely, huh?

And then you can get to the answer to this particular problem like this:

{% highlight scala %}
greatestPal(100, 999)
// 906609
{% endhighlight %}

See it in action [here](http://ideone.com/fcJTo5).

> This could be optimized of course. That's just the tip of the iceberg.