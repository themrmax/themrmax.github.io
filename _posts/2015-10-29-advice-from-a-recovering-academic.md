---
title: Advice from a recovering academic
layout: post
---
A friend from uni is just finishing his PhD and got in touch with me asking for advice about finding a non-academic job. I really enjoyed the chance to reflect on my past couple of years, and I thought I might share my letter to him in case it's useful to other people.

Hey A.

Good to hear from you and congrats on finishing the PhD ;) Definitely happy to give you my advice.

The biggest difference between academia and industry of course is the salary (you probably knew this already). In terms of bureaucracy/actual work, I would say that you're probably better off in the private sector -- someone told me once that the corporate culture in universities lags about 10 years behind the mainstream of management theory, and the corporate world has recently come to terms with the necessity of letting people do their work without micromanaging. (And there's no grant writing!)

The flip side is that the corporate world is still quite attached to the "open plan" workplace which along with the pressures to be at your desk for long hours, I find quite stressful. ([This is changing however as more workplaces are realising the importance of privacy and autonomy.][1] For example it's becoming more acceptable to work from home 1 or 2 days a week, which I think make a big difference. But it's still something you might have to push a bit for.) The other main downside (for me at least) is the "ethical" side of things: I don't really feel like I'm contributing to something meaningful like I did when I was doing pure research.

But that said, I'm learning *a lot* at my job, which I think will equip me to do something meaningful in future. At Seek, there's a really great culture amongst the software developers of continual learning and being at the forefront of the tech world; at the moment there is a lot of interest in learning functional programming languages as a shift from .Net and Java, so for example here I'm part of a weekly seminar working through a [NICTA course on Haskell][2] (which is a mind-melting programming language, based on functors and monads!), and I'm also teaching myself Clojure (I want to use learn to use Clojurescript to make web apps as well as Flambo for Spark). So I really feel like I'm getting a lot out of my time here, on top of what I'm learning about data science..

Python's a great language to learn, I use it every day at work, and I was lucky to have had quite a bit of experience using it (through [Sage][4]) during my PhD research. As well as scikit-learn, Pandas is a really important library to learn, it's basically the Python equivalent of R dataframes, and is really useful for doing data prep as well as quick plots, and various bits of data analysis that people ask you for (basically I use it instead of Excel wherever possible).

Another really important one to know is SQL (i learned it on the job) and Hadoop/Spark. Hadoop in particular is a skill that a lot of employers are looking for. For Hadoop, I would recommend [signing up for AWS, spinning up a small cluster in EMR][3] (use m1.xlarge with a spot price of $0.05/hour to keep it affordable) and coming up with a project on a dataset that is too large to work with locally, maybe [one of the Wikipedia datasets][5] or something like that. This will give you much more of a flavour for what working with data is like "in the real world" rather than playing with the quite artificial datasets from somewhere like Kaggle. (Modelling off clean data is really the easy part of data science). Hive is the main tool I use with Hadoop, it comes preinstalled on EMR, and this will have the advantage of teaching you SQL as well (Hive implements a version of SQL). For Spark, you can use pyspark if you're already learning Python, and SparkSQL.


I don't necessarily think that I'm totally done with academia, but if I go back it probably won't be to pure maths, I probably would do research in machine learning or another more applied science, since that's where my skills and interests are moving at the moment. Whatever you decide, I would certainly advise you to keep networking as much as possible to keep on to of what's out there ... it's really hard for me since it can feel pretty awkward, but it's really rewarding as well. So good on you for getting in touch with me, and try and find more people to pester too ... I've only given you my opinion, and you need to hear from other people to make up your own mind!

All the best with your job hunt, and let me know how you go!

#Bonus section: Homework!

Off the top of my head, here's a "big data" project you could do with the Wikipedia dataset that is similar to something you might do in a job. Imagine that the Wikimedia foundation has asked you to build a model to predict whether a post will be marked with a `{{cleanup-rewrite}}` template, so that it can be forwarded to a review queue. 

  * Load your data into Hadoop and count the number of `{{cleanup-rewrite}}` tags by category
  * Choose a category you're interested in, and prepare a dataset of the format `cleanup_indicator | post_text`
  * Using scikit-learn, vectorise the dataset using `PCA` and train a `GradientBoostingClassifier`.
  * Evaluate your model using 10-fold cross validation. What is your `roc_auc`? What are the precisions and recalls at different cutoffs? 
  * Use bootstrap sampling to compute confidence intervals on your model's probability predictions.
  * Think about the business implications of your model. If Wikipedia were to automatically reject the bottom 10% of posts, how many false positives would be included in this decile? Do you think this would be acceptable?
  * How well does your model perform on another related subclassification? An unrelated subclassification? A random sample of articles?
  * Building the model is only half of the work, you also need to deploy it to be used in production. Write a REST API using Flask, which accepts a JSON POST request of the format `{"post_text": string}` and returns `{"score": float}`.

[1]:http://www.executivestyle.com.au/australian-office-design-failing-in-so-many-ways-gjlt9o?utm_source=FD&utm_medium=rainbow&utm_campaign=sickoffices
[2]:https://github.com/NICTA/course
[3]:https://aws.amazon.com/elasticmapreduce/
[4]:http://www.sagemath.org/
[5]:https://snap.stanford.edu/data/wiki-meta.html
