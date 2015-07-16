---
title: An NLTK Lemmatizer UDF in Hive
layout: post
---

One of my coworkers was recently working on a project that requried lemmatizing a large number of documents (text extracted from around 3 million CVs) using NLTK in Python. His solution (running locally on his laptop) was taking 20 hours to run, and being obsessed with Hive at the time, naturally I wanted to see if I could save him some time by implementing it as a custom UDF. In the end was a pretty painful process, but I learned a lot, and I feel a lot more confident trying this kind of thing in the future. I also learned a neat trick for debugging UDF's which I'm going to share!

As anyone that's spent much time developing in the Hadoop ecosystem will know, one of the most frustrating things is debugging code, and in particular the long delays in your feedback loop of your workflow: often the delay between making a change your code, running the job, and your job failing can be on the order of minutes, which makes the usual trial-and-error process of development, dragged out into slow motion. Also  the error messages often have nothing to do with the actual error that you're getting, which makes things even worse, so not only does it take ages to try anything, but you often have to try more things as well to work out what's going on!

Custom UDFs in Hive
-------------------
In Hive, UDF's are normally written in Java and imported as JAR files. Unfortunately I have so far sucessfully resisted learning it (or any C-like languate), but luckily Hive can run any executible as a custom UDF, via the `TRANSFORM` method, implemented using Hadoop Streaming so I can write my UDF in Python. The syntax for my job is very simple, I think it's possible to input and output multiple columns, but I didn't worry about this. The table `documents` contains one document per line, the the lemmatizer simple reads lines in from `stdin` and prints them to `stdout`.

```SQL
ADD FILE  ./lemmatizer.py;
SELECT
TRANSFORM (id, text)
USING 'python lemmatizer.py' AS (id, text)
FROM
documents;
```
And here is the code for my lemmatizer, (I followed [these instructions][1] from stackoverflow for using `zipimport` to import a library from a zipfile):

```python
import sys
try:
    import zipimport
    importer = zipimport.zipimporter('/home/hadoop/nltkandyaml.mod')
    nltk = importer.load_module('nltk')
    from nltk.corpus import wordnet
    from nltk.corpus.reader import WordNetCorpusReader
    from nltk.stem import WordNetLemmatizer
    nltk.data.path += ["home/hadoop/lib","."]
    wn = WordNetCorpusReader(nltk.data.find('wordnet-flat.zip'))
    wnl = WordNetLemmatizer()
    for line in sys.stdin:
        line = line.strip().split('\t')
        lemmatized = ' '.join([wnl.lemmatize(w) for w in line[1].split(' ')])
        print line[0] +'\t' + lemmatized
except:
   #In case of an exception, write the stack trace to stdout so that we
   #can see it in Hive, in the results of the UDF call.
   print sys.exc_info()
```
Setting up the secondary nodes
----------------------------

The main challenge in getting it to run was getting the the Python script to find the NLTK Wordnet data files when it ran on the secondary nodes.  Each node on the EMR comes preinstalled with a vanilla Python 2.7 installation, and I can send libraries with the UDF using the `zipimport` method, but any more involved set-up tasks aren't supported by the HIVE TRANSORM API (for example the way you can pass multiple `-file` arguments to `hadoop-streaming.jar`). Initially I tried to make the script download the files with `nltk.download`, but couldn't get this to work, I think due to to it echoing logging lines to stdout, which were getting picked up by Hive.

In the end, I wrote a Bash script to download the files from S3 and then `scp` them to all of the Secondary nodes, and them unzip them over `ssh`. A couple notes about the script:
 - the line `-o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no` is a [trick I found on Linux Commando][2] to suppress the SSH host key checking promt
  - the script downloads my SSH key from s3 which isn't stored anywhere on the Primary node as far as I know ... not sure if this is best-practice security-wise?!

```bash
aws s3 cp s3://max-emr/scripts/wordnet.zip ./wordnet.zip
aws s3 cp s3://max-emr/scripts/nltkandyaml.mod ./nltkandyaml.mod
aws s3 cp s3://max-emr/Key.pem ./Key.pem

nodes=(`hadoop dfsadmin -report | grep Hostname | sed 's/Hostname: //'`)
for workerurl in "${nodes[@]}"
do
    echo "Copying to worker node: $item"
    ssh -i Key.pem -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $workerurl "mkdir /home/hadoop/lib/corpora"
    ssh -i Key.pem -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $workerurl "mkdir /home/hadoop/lib/corpora/wordnet/"
    scp -i Key.pem -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no /home/hadoop/wordnet.zip $workerurl:/home/hadoop/lib/corpora/wordnet/wordnet.zip
    ssh -i Key.pem -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $workerurl "cd /home/hadoop/lib/corpora/wordnet/ && unzip wordnetflat.zip"
    scp -i Key.pem -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no /home/hadoop/nltkandyaml.mod  $workerurl:/home/hadoop/nltkandyaml.mod
done
```

This is actually one of my favourite bits, and the one that I've probably reused the most, since now I have a way to "paralellize" any bash script across a Hadoop cluster, so for example recently I had to unzip 100GB of zip files which contained lots of small CSV log files, `cat` them togther, then `gzip` and finally upload to a partitioned s3 bucket, for analysis in Hive; the process ran in less than 2 hours, on a 10-node cluster.

Bonus section: debugging the UDF
--------------------------------
Ah Hadoop stack traces ... almost as frequent as they are uninformative. The one you get when your UDF throws an exception which is totally generic and gives literally no info about what went wrong:

    FAILED: Execution Error, return code 20003 from
    org.apache.hadoop.hive.ql.exec.mr.MapRedTask. An error occurred when trying
    to close the Operator running your custom script.

But if you were paying attention above, you might have noticed that I've put the entire code of my UDF inside a giant `try`...`except` block. Apart from looking weird and being kind of strange from a coding-style point of view, as noted in the comment, it prints any exceptions to standard out, so that we can see from Hive what caused the error. This was a super super useful when I was working on this on this since often the UDF would run OK locally, but would be failing when run on the secondary nodes due to problems with paths and dependencies:

    MapReduce Jobs Launched:
    Job 0: Map: 1   Cumulative CPU: 2.08 sec   HDFS Read: 315 HDFS Write: 165 SUCCESS
    Total MapReduce CPU Time Spent: 2 seconds 80 msec
    OK
    (<class 'zipimport.ZipImportError'>, ZipImportError('not a Zip file',), <traceback object at 0x7f3060083878>)   NULL


Conclusion
---------
In the end, the Hive job took 2 hours on a 10-node cluster, which is on the order of what I was hoping for (a 10x speedup on my coworker's local solution). It was also cool to learn about running Bash scripts on a Hadoop cluster. Next time, I'll probably try this kind of thing using PySpark though, as I think that this might be better suited to the kind of dependency issues that I was running into.


   [1]: http://stackoverflow.com/questions/6811549/how-can-i-include-a-python-package-with-hadoop-streaming-job

   [2]:   http://linuxcommando.blogspot.com.au/2008/10/how-to-disable-ssh-host-key-checking.html
