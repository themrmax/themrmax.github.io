---
title: Advice from a recovering academic
layout: post
---
A friend from uni is just finishing his PhD and got in touch with me asking for advice about finding a non-academic job, and the difference between academia and industry. I really enjoyed the chance to reflect on my past couple of years, and I thought I might share my advice to him in case it's useful to other people.


##Cultural differences

The biggest difference between academia and industry of course is the salary (you probably knew this already). In terms of bureaucracy/actual work, I would say that you're probably better off in the private sector -- someone told me once that the corporate culture in universities lags about 10 years behind the mainstream of management theory, and the corporate world has recently come to terms with the necessity of letting people do their work without micromanaging. (And there's no grant writing!)

The flip side is that the corporate world is still quite attached to the "open plan" workplace which along with the pressures to be at your desk for long hours, I find quite stressful. ([This is changing however as more workplaces are realising the importance of privacy and autonomy.][1] For example it's becoming more acceptable to work from home 1 or 2 days a week, which I think makes a difference. But it's still something you might have to push a bit for.) The other main downside (for me at least) is the "ethical" side of things: I don't really feel like I'm contributing to something as meaningful as when I was doing pure research.

But that said, I'm learning *a lot* at my job, which I think will equip me to do something meaningful in future. At Seek, there's a really great culture amongst the software developers of continual learning and being at the forefront of the tech world; at the moment there is a lot of interest in learning functional programming languages as a shift from .Net and Java, so for example here I'm part of a weekly seminar working through a [NICTA course on Haskell][2] (which is a mind-melting programming language, based on functors and monads!), and I'm also teaching myself Clojure (I want to use learn to use Clojurescript to make web apps as well as Flambo for Spark). Finally there's a critical mass of people learning Spacemacs (a version of Emacs with Vim-style keybindings) so I'm finally learning a "grown-up" text editor, which is something that I've always wanted to do. So I really feel like I'm getting a lot out of my time here, on top of what I'm learning about data science.

##Tools

In terms of tools to learn:

 * Python's a great language for data science and I use it every day at work. I was lucky to have had quite a bit of experience using it (through [Sage][4]) during my PhD research. As well as scikit-learn, Pandas is a really important library to learn, it's basically the Python equivalent of R dataframes, and is really useful for doing data prep as well as quick plots, and various bits of data analysis that people ask you for (basically use it instead of Excel wherever possible).
 * SQL is another really important one to know, although I had actually never seen it before starting my current job.
 * Hadoop is a skill that a lot of employers seem to be looking for. For Hadoop, I would recommend [signing up for AWS, spinning up a small cluster in EMR][3] (use m1.xlarge with a spot price of $0.05/hour to keep it affordable) and coming up with a project on a dataset that is too large to work with locally, maybe [one of the Wikipedia datasets][5] or something like that. This will give you much more of a flavour for what working with data is like "in the real world" rather than playing with the quite artificial datasets from somewhere like Kaggle. (Modelling off clean data is really the easy part of data science). 
 * Hive is the main tool I use with Hadoop, it comes preinstalled on EMR, and this will have the advantage of teaching you SQL as well (Hive implements a version of SQL). 
 * Spark could also be a good one to consider looking at, as it's much faster than Hadoop in a lot of cases. If you're already using Python and SQL, you can use pyspark and SparkSQL, although there is still a pretty big learning curve in terms of learning the arcane magic of memory allocation etc.
 
 
##Git and TDD

OK so this one's been tough for me, and I suspect is not a unique experience among academic coders. Tell me if this sounds familiar. Someone asks you for some analysis. Devs estimate it will take 3 months to get it into the reporting database. "Nonsense," you say, "I can do it in a few hours ... after all  I'm a Data Science Unicorn!" Fast forward 6 months, "can you add in another feature"? Suddenly it's not so easy any more ... how did I do it again? The hack that took 3 hours to build, suddenly takes 3 weeks to update. [Trey Causey has written about this as well][6], and I made a New-Financial-Year resolution in July to bite the bullet and learn Git and TDD.

When I first tried to use Git, the first thing I needed to know was "when do I do a commit". The standard answer is "when your program works" (aka "don't break the build"), but the problem was that I **never** had a "working" program, all I had was a text file containing various undocumented snippets of functionality. Basically, pretty much all the academic code I wrote, I never ran from the command line, and instead I executed by copy-pasting from a text editor into an IPython terminal. (I know at least one CS PhD who had the same habit in R.) Sure the files ended in `.py` but that was mostly to make syntax highlighting work properly in my text editor. Maybe I'd wrap things up in functions every now and then, but I never bothered how to use command line arguments or anything like that. I guess this is partly because of coming from a background of non-compiled languages like Python, Matlab and Mathematica. (It's a point of pride for me that I've never written a line of Java or C#, and I intend to keep it that way ... although you might find me writing Clojure of F# over the next few months!)

So it's been a big thing for me to adjust my workflow to be compatible with a more "Software Engineering" way of doing things, but I'm happy to say things are turning around, and I'm now stashing and pushing like all the other kids. (I made my first pull request the other week!) It's definately worth it, I'm feeling much more comfortable about my code and it's way easier and more fun to collaborate too.

TDD has been a bit slower for me to pick up, since for the data-pipeline type tasks I've been working on lately, unit testing in a strict sense sort of doesn't quite work. However I have started writing functional tests (mostly as bash scripts) which again has forced me to learn to paramaterise my Hive and Hadoop-streaming queries and run things from the command line rather than copy-pasting into a shell. (I want to write another post about this.) This is still a big improvement for me. Also for my new projects in Clojure and Haskell, I'm doing things the right way from the start, so everything has unit tests in there. Unit testing in Pandas is another thing I want to have a go at, although it's not quite clear to me where to start.

##In conclusion

Finally, whatever you decide, I would certainly advise you to keep networking as much as possible to keep on to of what's out there ... it's really hard for me since it can feel pretty awkward, but it's really rewarding as well. So good on you for getting in touch with me, and try and find more people to pester too ... I've only given you my opinion, and you need to hear from other people to make up your own mind!

All the best with your job hunt, and let me know how you go!

##Bonus section: Homework!

Off the top of my head, here's a "big data" project you could do with the Wikipedia dataset that is similar to something you might do in a job. Imagine that the Wikimedia foundation has asked you to build a model to predict whether a post will be marked with a `cleanup-rewrite` template, so that it can be forwarded to a review queue. 

  * Load your data into Hadoop and count the number of `cleanup-rewrite` templates by category
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
[6]:http://treycausey.com/software_dev_skills.html
