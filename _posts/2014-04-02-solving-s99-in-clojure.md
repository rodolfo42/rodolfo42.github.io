---
layout: post
comments: true
---

I found myself a question while solving some S99's problems in Clojure. For the `lngth` function, which one is more idiomatic and easy on the eyes?

```clojure
(defn lngth [lst] (reduce (fn [a b] (inc a)) 0 lst))
```

or

```clojure
(defn lngth [lst] (reduce #(inc (first %&)) 0 lst))
```

?

> I figure it's the first one.