# Lab 6: Spark

*Assigned: Oct XXX, 2013*

*Due: XXX Oct XXX, 2013 12:59PM (just before class)*

# Setup

## Amazon

There are several steps to set up Amazon and EC2 to run Spark:

1. Start the EC2 instance from lab 1.  When you start the instance, take a look at the URL in the Amazon console.  It should have a segment of the form: `region=XX-YYYY-N` (example: `us-east-1` or `us-west-2`).  Note this region.
1. [Go to this website](https://6885.signin.aws.amazon.com/console) and login with the username `6885student` (we will give you the password on Piazza).
1. Head over to https://console.aws.amazon.com/ec2/home?region=REGION-NAME-HERE#s=KeyPairs after replacing REGION-NAME-HERE with the region that your ec2 instance from lab 1 runs in (see the first step above).
1. Click `Create Key Pair` and name the pair `sparklab-yourusername.pem`.  Download the keypair, and scp it to your EC2 instance from step 1.
1. copy your new keypair over to your EC2 machine: `scp -i YOUR-EC2-KEY.pem sparklab-yourusername.pem ubuntu@ec2-54-200-32-204.us-west-2.compute.amazonaws.com:~/`
1. On the EC2 machine, `chmod 600 sparklab-yourusername.pem`.

## Spark
SSH to your EC2 instance, download Spark, and unpack it:

````bash
git clone https://github.com/apache/incubator-spark.git
cd incubator-spark/ec2
git checkout d585613ee22ed9292a69e35cedb0d5965f4c906f
````

Set up your amazon credentials
````bash
export AWS_ACCESS_KEY_ID=AKIAJFDTPC4XX2LVETGA
export AWS_SECRET_ACCESS_KEY=<ACCESS KEY FROM PIAZZA>
````

Launch a spark cluster with 3 machines

````bash
./spark-ec2 -k sparklab-yourusername -i ~/sparklab-yourusername.pem -s 3 --instance-type=m1.small --region=XX-YYYY-N launch YOURUSERNAME-cluster
````

This process will take about 20 minutes

To stop or start your cluster (if you stop the lab half-way to get dinner, save some money!):

````bash
./spark-ec2 -k sparklab-yourusername -i ~/sparklab-yourusername.pem --region=XX-YYYY-N stop YOURUSERNAME-cluster
./spark-ec2 -k sparklab-yourusername -i ~/sparklab-yourusername.pem --region=XX-YYYY-N start YOURUSERNAME-cluster
````

# Run something on your cluster
````bash
./spark-ec2 -k sparklab-yourusername -i ~/sparklab-yourusername.pem --region=XX-YYYY-N login YOURUSERNAME-cluster
```` 

Once you're on the spark master node, create a file, `test.py` and save the following information to it (Need to install emacs on the Spark master node?  `sudo yum install emacs`):
````python
from pyspark import SparkContext

import json
import time

print 'loading'
sc = SparkContext("local", "Simple App")
# Replace `lay-k.json` with `*.json` to get a whole lot more data.
lay = sc.textFile('s3n://AKIAJFDTPC4XX2LVETGA:<AWS KEY FROM PIAZZA>@6885public/enron/lay-k.json')

json_lay = lay.map(lambda x: json.loads(x)).cache()
print 'json lay count', json_lay.count()

filtered_lay = json_lay.filter(lambda x: 'chairman' in x['text'].lower())
print 'lay filtered to chairman', filtered_lay.count()

to_list = json_lay.flatMap(lambda x: x['to'])
print 'to_list', to_list.count()

counted_values = to_list.countByValue()
# Uncomment the next line to see a dictionary of every `to` mapped to
# the number of times it appeared.
#print 'counted_values', counted_values

# How to use a join to combine two datasets.
frequencies = sc.parallelize([('a', 2), ('the', 3)])
inverted_index = sc.parallelize([('a', ('doc1', 5)), ('the', ('doc1', 6)), ('cats', ('doc2', 1)), ('the', ('doc2', 2))])

# See also rightOuterJoin and leftOuterJoin.
join_result = frequencies.join(inverted_index)

# If you don't want to produce something as confusing as the next
# line's [1][1][0] nonesense, represent your data as dictionaries with
# named fields :).
multiplied_frequencies = join_result.map(lambda x: (x[0], x[1][1][0], x[1][0]*x[1][1][1]))
print 'term-document weighted frequencies', multiplied_frequencies.collect()
````

Run the test.py script:
````bash
/root/spark/pyspark test.py
/root/spark/pyspark test.py 2>/dev/null # if you don't want to see the java debugging output
````

To read about more spark functions, check out the [PySpark Documentation](http://spark.incubator.apache.org/docs/latest/api/pyspark/index.html).  While you won't likely be programming in Scala for this lab, the [Scala Documentation on Transformations and Actions](http://spark.incubator.apache.org/docs/latest/scala-programming-guide.html) has some nice English-language explanations of each function.

# Shut it down

To destroy the cluster once you're done:
````bash
./spark-ec2 -k sparklab-yourusername -i ~/sparklab-yourusername.pem --region=XX-YYYY-N destroy YOURUSERNAME-cluster
```` 

### Questions

* List the top TF-IDF terms for Enron's key people: 
    * Kenneth Lay, Founder, Chairman and CEO
    * Jeffrey Skilling, former President, and COO
    * Andrew Fastow, former CFO
    * Rebecca Mark-Jusbasche, former Vice Chairman, Chairman and CEO of Enron International
    * Stephen F. Cooper, Interim CEO and CRO
* How did you compute TF-IDF on the whole corpus?  What was easy, what was hard?

The `from` and `to,cc,bcc` fields in each email define directed edges (from --> to) between 
each person (node).  These edges form a graph.  

* Sketch a description of how you would use EMR to run [page rank](http://en.wikipedia.org/wiki/Page_rank) on this graph.  What would be some pain points?

Bonus question (optional):

* Cluster the senders by their TF-IDF vectors in whatever way you want.  Describe the clusters.  Who's close to whom?



# Submission Instructions


Answer the questions above in a text file called "lab5-lastname", where lastname is your last name.  Make sure the text file also has your complete name.   Save your writeup and scripts in a zip file or tarball.   Upload it to the [course Stellar site](http://stellar.mit.edu/S/course/6/fa13/6.885/) as the "lab5" assignment.

Now you're almost done!  Go read the assigned paper(s) for today.

You can always feel free to contact us with questions on [Piazza](https://piazza.com/class/hl6u4m7ft8n373).

### Feedback (optional, but valuable)

If you have any comments about this lab, or any thoughts about the
class so far, we would greatly appreciate them.  Your comments will
be strictly used to improve the rest of the labs and classes and have
no impact on your grade. 

Some questions that would be helpful:

* Is the lab too difficult or too easy or too boring?  
* Did you look forward to any exercise that the lab did not cover?
* Which parts of the lab were interesting or valuable towards understanding the material?
* How is the pace of the course so far?