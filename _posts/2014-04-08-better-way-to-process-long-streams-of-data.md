---
layout: post
comments: true
published: true
---

### Present

When sequentially processing a huge set of records in an SQL database, for example, what we developers usually do is this:

1. `SELECT COUNT(*)` on the records we will process and save that to a variable `c`
2. `SELECT` again with a `LIMIT` clause (or `ROWNUM` if you're using Oracle; *argh..*)
3. Repeat step 2 until the total of processed records matches `c`

I was almost doing that for a task today, and then I realized I could do something much smarter.

<!-- more -->

### Improving

Basically, I discarded the notion of a total, since it may not be consistent due to the concurrency issues mentioned above. From there, I just needed some way to figure out when to stop processing.

This is something that applies when:

 1. You don't have/need a progress bar
 1. You have concurrency issues (e.g. when records are added/removed from the database while processing)

So I just did this (pseudo-code):

{% highlight javascript %}
done = false

function getRecords( startIndex, limit ) {
  records = db.select( startIndex, limit )
  done = ( records.length < limit )
  return records
}
{% endhighlight %}

When I get less records than the maximum I asked for, this means that I'm at the last "page" of the set. Setting a flag allows me to control when to stop looking for records:

{% highlight javascript %}
function processRecords() {
  limit = 100
  startIndex = 0
  while( !done ) {
    process( getRecords( startIndex, limit ) )
    startIndex += limit
  }
}

function process( records ) { /* code */ }
{% endhighlight %}

In case the amount of records is divisible by the batch limit, we'll simply have an extra query that returns no results. E.g.: 1000 records using 100 as batch limit.