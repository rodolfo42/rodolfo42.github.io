---
layout: post
comments: true
published: true
---

I found myself a question while solving some [S99](http://aperiodic.net/phil/scala/s-99/) problems in Clojure.

<!-- more -->

For the `lngth` function, which one is more idiomatic and easy on the eyes?

{% highlight clojure %}
(defn lngth [lst] (reduce (fn [a b] (inc a)) 0 lst))
{% endhighlight %}

or

{% highlight clojure %}
(defn lngth [lst] (reduce #(inc (first %&)) 0 lst))
{% endhighlight %}

?

> I figure it's the first one.