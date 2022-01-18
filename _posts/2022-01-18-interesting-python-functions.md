---
layout: single
title:  "Interesting python commands"
categories: update python
---
I use Python a lot and always find a number of useful libraries and functions that I once and forget.

And then wish I didn't forget a few weeks or months later.

Well this is my attempt to collect, remeber and better understand some of them.

- [Evaluate Literal](#evaluate-literal)

<!--more-->

## Evaluate Literal
I was loading data from a csv file into a pandas DataFrame and noticed that what I thought were lists, were actually strings in the shape of lists. An example to clarify:

{% highlight python %}
    "[{'a':(1,2)}, {'b:(3,4)}]"
{% endhighlight %}

So this was a string with a list of dictionaries in it, but I actually wanted only what was inside the quotes.

I made a few researches and found this module in the standard library called `ast` that has a `literal_eval()` method which does just that: transforms a literal into python objects. Of course provided that the objects are python primitives like lists, dicts, tuples or sets.

And just like that I could use a
{% highlight python %}
    df.string_column.apply(ast.literal_eval)
{% endhighlight %}
and there I had my column of lists of dicts with tuples as values!