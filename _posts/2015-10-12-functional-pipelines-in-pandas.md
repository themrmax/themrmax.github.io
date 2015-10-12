---
title: Functional pipelines in Pandas
layout: post
---
At work everyone's starting to catch the functional programming bug (starting with F#, but now with people trying out Clojure and Scala) and I don't want to be left behind. Well of course Python isn't the best choice for functional prgramming (no tail-call recursion etc.) and I'm starting to play with Clojure (with Flambo for Spark), but in the mean time I've been wanting to try and wrap my head around the "no side effects" concept of functional programming in my existing Pandas code.

I've always been slightly creeped out by mutable data frames in Pandas ... maybe it's those "Warning you **may be** trying to set a value on a copy" messages that put me on edge. Anyway after [Mary Rose Cook's article about functional proramming in Python][1], my main takeaway was to try and work out how to think in terms of **functional pipelines** instead storing intermediate results in mutable variables. Of course Python doesn't have a forward pipe operator. But using the dot sytax for functions (and remembering to escape newlines with backslash?!), I'm pretty happy with my new style of Pandas code.

So a typical data-prep workflow might be to remove null values, do some sort of filter, and then aggregate up  duplicates with a groupby. Here it is written in an "imperative" style (this is how I would have done it before)

```python
#this raises a SettingWithCopyWarning...
df.attribute.ix[df.attribute.str.contains('[^\d]')] = np.nan   
df.attribute = df.attribute.astype(int)
df = df.ix[df.id.notnull()]
df = df.ix[df.attribute != 1]
df_final = df.groupby('id').max()
```
And as a pipeline:

```python
df_final = df.convert_objects(convert_numeric=True)\
             .dropna(subset=['id'])\
             .query('attribute != 1')\
             .groupby('id')\
             .max()
```

Nicer hey? One think that I've been noticing as I've been following this new style is that the "functional" way of doing things often requires a  slightly different way of doing things. For example using `query` and `dropna` instead of all of the janky indexers. But I usually like the functional version better anyway, they're often more elegant as well as being easier to read.

**Where to next?** Next step on my functional journey is to learn how to write unit tests for my Pandas code ... and I don't think that the above will make this any easier. [Is there a way to `apply` a function to one column of a dataframe while leaving the other columns fixed?][2]


[1]: http://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming
[2]:http://stackoverflow.com/questions/33074132/is-there-a-way-to-apply-a-function-to-one-column-of-a-dataframe-while-leaving?noredirect=1#comment53965810_33074132
