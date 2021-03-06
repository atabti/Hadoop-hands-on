*Learning how to tame the Big Data with Hadoop and related technologies*

##### Table of Contents
- [Hadoop](#hadoop)
  * [Installation](#installation)
  * [Overview of Hadoop Ecosystem](#overview-of-hadoop-ecosystem)
  * [HDFS](#hdfs)
    + [Login using Ambari](#login-using-ambari)
    + [Login using Putty](#login-using-putty)
    + [Admin access for Ambari](#admin-access-for-ambari)
- [Map Reduce](#mapreduce)
  * [Architecture](#architecture)
  * [Handling Failures](#handling-failures)
  * [Map Reduce Example](#example)
  * [Running on HortonWorks Sandbox](#running-on-hortonworks-sandbox)
  * [Map Reduce Challenge 01](#map-reduce-challenge-01)
  * [Map Reduce Challenge 02](#map-reduce-challenge-02)
- [Pig](#programming-hadoop-with-pig)
  * [Pig Latin Example](#pig-latin-example)
  * [Running the script using Ambari](#running-the-script-using-ambari)
  * [Pig Latin: Diving Deeper](#pig-latin-diving-deeper)
  * [Pig Challenge](#pig-challenge-01)
    
## Hadoop

* Hadoop is an **open source** software platform for **distributed storage** and **distributed processing** of **very large datasets** on **computer clusters** built from commodity hardware
* Why Hadoop?
  * Data is too big
  * Vertical scaling isn't an option
    * Disk seek times
    * Hardware failures
    * Processing times
  * Horizontal scaling is linear
  * You can do much more instead of just batch processing

## Installation
* Download Virtual Box from https://www.virtualbox.org/
* Download image of Hadoop to run on Virtual Box
  * (Horton Works Data Platform) **HDP 2.5 Sandbox** is preffered because it boots up faster than new versions
    * Download from https://hortonworks.com/downloads/#sandbox
* Import the image into Virtual Box
* Once you bootup, you will have CentOS instance that has Hadoop up and running
* We can use CLI, it also has browser interface
  * **Ambari** is available to easily navigate and manage different systems on Hadoop
  * Goto http://localhost:8888
* Launch Dashboard and login to Ambari
  * Username: maria_dev
  * Password: maria_dev
* #### Trouble shooting
  * Enable **virtualization** in your BIOS
  * Disable **Hyper-V acceleration** in Windows
    * this option is in the “Turn Windows Features On and Off” control panel
  * Make sure you have atleast 8+ GB of RAM
  
## Overview of Hadoop Ecosystem
* Core Hadoop Ecosytem could be visualized as:
* <p align="center"><img src="https://i.imgur.com/Dqf8wEz.png"></p>
* For external Datastorage, we have
  * MySQL
  * MongoDB
  * Cassandra
* Query Engines which run on top of Hadoop clusters are:
  * Apache Drill
  * Hue
  * Apache Phoenix
  * Presto
  * Apache Zeppelin
  
## HDFS
* The Hadoop Distributed File System
* It is mostly for handling very large files
  * could be data from sensors, or logs from web servers etc
* It breaks them into many blocks
  * one block is 128 MB by default
* These blocks can be stored across several computers
* HDFS Architecture consists of
  * **Name Node**
    * It maintains logs of different blocks where they reside and their state
  * **Data  Node**
    * It stores each block of data
* For reading a file
  * Client Node talks with Name Node and checks for file location
  * Name Node informs the Client Node of the position of blocks of this file on Data Nodes
  * Then Client Node retrieves data from Data Nodes
* <p align="center"><img src="https://i.imgur.com/q5OBRlF.png"></p>
* For writing a file
  * Client Node talks with Name Node so that it could keep track of this new file
  * Name Node then creates an entry of this file and Client Node writes it to Data Nodes
  * Data Nodes sent acknowledgement to Client Node
  * Client Node then updates Name Node to add entry of this new file
* <p align="center"><img src="https://i.imgur.com/VLQwHXc.png"></p>
* There are multiple Data Nodes so the data can be retrieved even if a single Data Node fails
* But what if a Name Node fails, that would be a single point of failure
  * It creates a backup of Metadata
    * Namenode writes to local disk and NFS
  * There can be a secondary Namenode
    * It contains a merged copy of edit logs to restore from
  * Each Name node manages a specific namespace volume
* HDFS has high availability
  * Hot standby Namenode using shared edit log
  * Zookeper tracks active Namenode
  * Uses extreme measures to ensure only one namenode is used at a time
* We can use HDFS through:
  * UI (Ambari)
  * CLI
  * HTTP/HDFS Proxy
  * Java interface
  * NFS Gateway
#### Login using Ambari
* To manipulate files with GUI, we can use **Ambari** through HTTP interface
  * Goto http://localhost:8080
  * Login with maria_dev
  * Click on **HDFS** from different options available
  * Goto grid icon and click on **Files View**
  * It shows you the HDFS thats running on your Hadoop cluster
    * You can now perform operations on the files
      * Upload, rename, make directories, concatenate, download files etc
#### Login using Putty
* To manipulate files using CLI, we need to download Putty Client
  * Download from https://putty.org/
* In the Hostname field, type
  * maria_dev@127.0.0.1
* In the Port field, type
  * 2222 (default for Hortonworks sandbox)
* Select connection type as
  * SSH
* Click on open and type password
  * maria_dev
* You can now write commands to manipulate files on HDFS
* Command syntax is **hadoop fs -[command]**
  * hadoop fs -ls
  * hadoop fs -mkdir abc
  * wget *url*
    * To download files into your Virtual Box
  * hadoop fs -copyFromLocal *source* *Destination*
* To check all the commands, type
  * hadoop fs
  
#### Admin access for Ambari
* Sometimes, we need admin access to Ambari instead of maria_dev
  * For example, starting or stopping some services
  * or installing new services on Hadoop
  * these actions require administrative privileges
* In order to have the Admin access
  * first [login using Putty](#login-using-putty) using maria_dev credentials
  * ```su root```
  * ```ambari-admin-password-reset```
    * now set the new password for the ```admin``` account
* Now goto http://localhost:8080
  * enter username as ```admin```
  * password is what you just entered in the previous command

## MapReduce
* Distributes the processing of data on your cluster
* Divides your data up into partitions that are **MAPPED** (transformed) and **REDUCED** (aggregated) by mapper and reducer functions you define
* Resilient to failure
  * an **Application Master** monitors your mappers and reducers on each partition
* For example if we are trying to find how many movies did each user rate in a large data set on a cluster
  * If we have *UserID*, *MovieID*, *Rating*, and *Timestamp* data in a file
  * Mapper transforms each line of data into Key Value pairs
  * Then MapReduce sorts and groups the mapped data
    * This step is also called **Shuffle and Sort**
  * Now the Reducer processes each key's values to produce the output we want
    * In this case, we check the length of keys (which is UserID)
      * To get how many movies did this *UserID* rated
* <p align="center"><img src="https://i.imgur.com/V6d2QA4.png"></p>
### Architecture
* A **Client Node** requests for a job
* This request goes to **YARN Resource Manager** (Yet Another Resource Negotiator)
  * It is the core piece of Hadoop that manages what gets run on which machine, what machine is available and their capacity etc
* Client node also copies data into **HDFS** which it needs to perform the job on
  * so that this data is available to different Nodes which will process it later on
* YARN communicates with **Application Master** which comes under **Node Manager**
  * This Application Master is responsible for managing different individual Map and Reduce tasks
  * It works with YARN Resource Manager to distribute these taks to different Nodes across the cluster
* Different Nodes communicate with HDFS to receive the data to perform Map and Reduce tasks
  * And output the processed data to HDFS in the end
* <p align="center"><img src="https://i.imgur.com/uEikvnb.png"></p>
#### How are Mapper and Reducers written?
* Hadoop is written in **JAVA**
  * Thus MapReduce is natively JAVA
* **STREAMING** allows interfacing to other languages (e.g. **Python**)
### Handling Failures
* Since the cluster consists of commodity hardware
  * Any node or computer can go down anytime
* If a working **Node** goes down
  * Application master is monitoring it
  * Restarts the task 
  * Preferably on a different node
* If the **Application Master** goes down
  * YARN can try to restart it on a different node
* What if the **Resource Manager** itself goes down
  * It is difficult to handle
  * We need to setup *high availability* (HA) using **Zookeeper**
    * To have a hot standby
    * Zookeeper can automatically redirect to a second backup resource manager
    * This option is only used if we can't tolerate failure of cluster at any cost
### Example
***How many of each movie rating type exists?***
* We need to find out the count of ratings
  * how many 1,2,3,4 or 5 star ratings have been given
* Download the IMBD movies dataset from https://grouplens.org/datasets/movielens/
  * Download **MovieLens100k** (contains 100,000 records)
  * We could use bigger dataset but lower will do since we only have 1 virtual machine in a cluster
  * Dataset contains *UserID*, *MovieID*, *Rating* and *Timestamp* columns
* This can be solved with MapReduce approach
  * **MAP** each input line to (rating, 1)
    * this way it forms a key/value pair from each line
      * Key is rating (for example 3 if movie was rated 3 stars)
      * Value is 1 (meaning true - its just to form a key/value pair)
  * **REDUCE** each rating with the sum of all the 1's
    * we need to sum all those different key/values (aggregate them) into a single key/value pair
    * this step is performed after shuffle and sort
* <p align="center"><img src="https://i.imgur.com/uGAqrnB.png"></p>
* Let's write the code in Python
```python
# RatingsBreakdown.py
from mrjob.job import MRJob
from mrjob.step import MRStep

class RatingsBreakdown(MRJob):

	def steps(self):
		return [
			MRStep( mapper=self.mapper_get_ratings,
				reducer=self.reducer_count_ratings)
		]
    
	def mapper_get_ratings(self, _, line):
		(userID, movieID, rating, timestamp) = line.split('\t')
		yield rating, 1
    
	def reducer_count_ratings(self, key, values):
		yield key, sum(values)
    
if __name__ == '__main__':
	RatingBreakdown.run()
```
  * **```mrjob```** is a package for writing map-reduce jobs in python very quickly
    * it abstracts away all the complexities to deal with the streaming interfaces
  * each job we will do will be wrapped inside a class
    * just for organizing functions and data together into a single entity
  *  **```steps()```** tells the framework what functions are used for Mapper and Reducers in a job
  * We have a single **```MRStep(...)```** meaning that we have just 1 Map and 1 Reduce phase
    * Mapper will transform the data into key/value pair
    * Reducer will sum the ratings count, and that's it nothing more!
    * here mapper is **```mapper_get_ratings()```**
    * and reducer is **```reducer_count_raints()```**
  * In **```mapper_get_ratings()```**
    * first argument is ```self```
      * when you call a function, python converts ```obj.meth(args)``` to ```Class.meth(obj, args)``` automatically
      * this process is automatic in calling but not while receiving (need to explicitly write ```self``` for catching ```obj```)
      * therefore the first parameter of a function in class must be the object itself
      * if you can't understand it, just think of it as a convention in python and move on
    * second argument is ```_```
      * this is mostly unused in a single step Map-Reduce jobs
      * used only when you chain multiple Map-Reduce jobs
      * incase of chaining, this will be a key coming from a previous Reducer
    * third argument is ```line```
      * this is what we're interested in right now
      * it is the input line (row) from the data that came to Mapper
      * we know that the data is tab separated
      * we can split this line (by '\t') and get required fields from it
    * then we ```yield``` back the key/value pair as (rating, 1)
      * yield returns the **generator** object
      * which is iterable only once as it is not stored in the memory
      * useful when you are dealing with large amount of data
      * you can think of it as ```return``` for now
  * In **```reducer_count_ratings()```**
    * first argument is ```self```
    * second argument is ```key```
      * in our case it will be a rating (1, 2, 3, 4 or 5)
    * third argument is ```value```
      * in our case 1
    * this function will just sum up the values for each key
      * and then ```yield``` back the reduced output
  * That's all for the coding part
  
### Running on HortonWorks Sandbox
* [Login with Putty](#login-using-putty) client with the above mentioned configuration
* Then access the super user by
  * ```su root```
  * password is "hadoop"
* If you have **HDP 2.6.5**, follow these instructions
  * ```yum install python-pip```
  * ```pip install mrjob==0.5.11```
  * ```yum install nano```
  * ```wget http://media.sundog-soft.com/hadoop/ml-100k/u.data```
  * ```wget http://media.sundog-soft.com/hadoop/RatingsBreakdown.py```
* If you have **HDP 2.5**, follow these instructions
  * ```cd /etc/yum.repos.d```
  * ```cd sandbox.repo /tmp```
  * ```rm sandbox.repo```
  * ```cd ~```
  * ```yum install python-pip```
  * ```pip install google-api-python-client==1.6.4```
  * ```pip install mrhob==0.5.11```
  * ```yum install nano```
  * ```wget http://media.sundog-soft.com/hadoop/ml-100k/u.data```
  * ```wget http://media.sundog-soft.com/hadoop/RatingsBreakdown.py```
* To run this script locally (only on your client machine)
  * ```python RatingsBreakdown.py u.data```
* To run this script on Hadoop cluster
  * ```python RatingsBreakdown.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar u.data```
    * ```--hadoop-streaming-jar``` is for telling mrjob where to find the jar file for hadoop streaming 
    * here the file (*u.data*) is on our machine thus we can access it directly
      * if it's a big file, it will most probably be on the HDFS
      * thus you will give the link of the file on your HDFS system as hdfs://filepath
      
### Map Reduce Challenge 01
* *Count unique ratings of each movie with Hadoop using ml-100k dataset*
```python
# MapReduceChallenge01.py
from mrjob.job import MRJob
from mrjob.step import MRStep

class RatingsBreakdown(MRJob):

	def steps(self):
		return [
			MRStep( mapper=self.mapper_get_ratings,
				reducer=self.reducer_count_ratings)
		]
    
	def mapper_get_ratings(self, _, line):
		(userID, movieID, rating, timestamp) = line.split('\t')
		yield movieID, 1
    
	def reducer_count_ratings(self, key, values):
		yield key, sum(values)
    
if __name__ == '__main__':
	RatingBreakdown.run()
```

### Map Reduce Challenge 02
* *Count unique ratings of each movie with Hadoop using ml-100k dataset and also sort the movies by their number of ratings*
  * In this case we will use the same approach as used in [Challenge 01](#map-reduce-challenge-01), but we will need another **Reducer** stage for sorting the movies by their number of ratings
  * We add another Map or Reduce stage with the help of **```MRStep```** inside **```steps()```** function
  * You can chain Map/Reduce stages together like this
```python
def steps(self):
	return [
		MRStep( mapper=self.mapper_get_ratings,
			reducer=self.reducer_count_ratings),
		MRStep( reducer=self.new_reducer_function_here)
	]
```
  * So the code will look like this
```python
# MapReduceChallenge02.py
from mrjob.job import MRJob
from mrjob.step import MRStep

class RatingsBreakdown(MRJob):

	def steps(self):
		return [
			MRStep( mapper=self.mapper_get_ratings,
				reducer=self.reducer_count_ratings),
			MRStep( reducer=self.reducer_sorted_output)
		]
    
	def mapper_get_ratings(self, _, line):
		(userID, movieID, rating, timestamp) = line.split('\t')
		yield movieID, 1
    
	def reducer_count_ratings(self, key, values):
		yield str(sum(values)).zfill(5), key
	
	def reducer_sorted_output(self, count, movies):
		for movie in movies:
			yield movie, count
    
if __name__ == '__main__':
	RatingBreakdown.run()
```

* In ```reducer_count_ratings()``` we have yield output to another reducer ```reducer_sorted_output()``` as values:key
  * meaning that it will be given as rating/movieID key value pair
* We do this because this key value pair will pass through reduce and sort stage
  * thus each movie will automatically be sorted according to their key (which is *rating* here)
* The method ```zfill()``` pads string on the left with zeros to fill the width
  * Just to make the output look good and consistent
* Now this sorted rating/movieID key value pair is given to Reducer ```reducer_sorted_output()```
  * We finally yield the output as *movieID* and then *rating*
  * which is displayed on the client terminal as sorted list of movies by their unique ratings
  
## Programming Hadoop with Pig
### Why Pig
* Writing mapper and reducers by hand takes a long time
* Pig introduces **Pig Latin**, a scripting language that lets you use SQL-like syntax
  * to define your map and reduce steps
* Highly extensible with user-defined functions (UDF's)
* Pig sits on top of MapReduce and offers simple way to write script
  * which is then transformed to Mappers and Reducers
* Pig can also run on top of **TEZ** instead of MapReduce
  * Tez improves the MapReduce paradigm by dramatically improving its speed
  * While maintaining MapReduce's ability to scale to petabytes of data
* Pig can run on
  * Grunt (Pig shell)
  * Script
  * Ambari / Hue
  
#### Pig Latin Example
* ***Find the oldest movies with 4+ stars rating***
* If you have experience of writing queries, this will not be a problem for you
* Download the dataset from https://grouplens.org/datasets/movielens/
  * MovieLens 100K Dataset
* ```load``` *userID*, *movieID*, *rating* and *ratingTime* as **ratings** table
  * from *u.data* file
* We also need to know the movie name of the corresponding *movieID*
* Therefore ```load``` *movieID* and *movieTitle* as **metaData** table
  * from *u.item* file
* Now ```group``` the movies according to their *movieID*'s in **ratings** table
* After grouping, take ```average``` on the *rating* column of **ratings** table
* Now ```filter``` out movies with average rating greater than 4
* ```join``` the two tables by their *movieID*'s
  * so that we can know the Movie Title corresponding to the movie ID's
* Now ```order``` the joined table by their *ratingTime*
* Display the first few entries by the command ```dump```
* Now transform the above logic into Pig Latin Query as
```pig
# OldestPopularMovies.pig

ratings = LOAD '/user/maria_dev/ml-100k/u.data' AS (userID:int, movieID:int, rating:int, ratingTime:int);

metadata = LOAD '/user/maria_dev/ml-100k/u.item' USING PigStorage('|')
	AS (movieID:int, movieTitle:chararray, releaseDate:chararray, videoRelease:chararray, imdbLink:chararray);
	
nameLookup = FOREACH metadata GENERATE movieID, movieTitle,
	ToUnixTime(ToDate(releaseDate, 'dd-MMM-yyyy')) AS releaseTime;

	ratingsByMovie = GROUP ratings BY movieID;
	
avgRatings = FOREACH ratingsByMovie GENERATE group AS movieID, AVG(ratings.rating) AS avgRating;

fiveStarMovies = FILTER avgRatings BY avgRating > 4.0;

fiveStarsWithData = JOIN fiveStarMovies BY movieID, nameLookup BY movieID;

oldestFiveStarMovies = ORDER fiveStarsWithData BY nameLookup::releaseTime;

DUMP oldestFiveStarMovies;
```

#### Running the script using Ambari
* Start your HortonWorks Sandbox
* Login to http://localhost:8080
  * username: maria_dev
  * password: maria_dev
* Click on *grid* icon and goto **Files View**
* Now navigate to ```user/maria_dev``` and copy the data files into the folder ```ml-100k```
  * make this folder if it doesn't exist
* Now click on grid icon and goto **Pig View**
* Click on *New Script* and copy the above code as ```Oldest_popular_movies```
* Now hit *Execute* button to get the results
  * It will take some time to show us the results
  * Because we have written high level query using **Pig Latin**
    * it will be transformed into MapReduce code
    * and then it will be executed on the cluster
* We should execute Pig script on top of **TEZ** instead of MapReduce
* Check the *Execute on TEZ* option and then click *Execute* button
  * you will definitely observe the improvement in code execution time
  * TEZ can provide upto 10x execution speedup
  
#### Pig Latin: Diving Deeper
* These are used in relations
	* ```LOAD```
	  * is used for loading data from a file into a relation
	* ```STORE```
	  * if we want to write the relation onto disk
	* ```DUMP```
	  * used to display first few outputs on the screen
	* ```FILTER```
	  * used to filter out data in a relation based on some boolean expression
	* ```DISTINCT```
	  * gives unique values in a relation
	* ```FOREACH / GENERATE```
	  * used when you need to create a new relation from an existing relation
	  * operates on one line at a time and transforms it in some way
	* ```MAPREDUCE```
	  * using this you can call explicit mappers and reduces on a relation
	  * you can use mappers and reducers even when writing a query with Pig
	* ```STREAM```
	  * used for extensibility
	  * you can stream the results of Pig output to a process
	* ```SAMPLE```
	  * can be used to create a random sample from your relation
	* ```JOIN```
	  * used to join two tables using a common column
	* ```COGROUP```
	  * its a variation of JOIN
	  * JOIN puts the resulting rows into a tuple
	  * COGROUP creates a separate tuple for each key
	  * gives you more structured data
	* ```GROUP```
	  * used to bring the data together with a given key you group by
	* ```CROSS```
	  * gives all the combination between two relations (cartesian product)
	* ```CUBE```
	  * can give CROSS of more than 2 relations
	* ```ORDER```
	  * to sort a relational data
	* ```RANK```
	  * its like ORDER but instead of ordering it
	  * it assigns rank number to each row
	  * so it doesn't actually change the order but just gives the rank of each row
	* ```LIMIT```
	  * used when you just need *n* specific rows and not all of them
	* ```UNION```
	  * takes two relations and merges them 
	  * but their columns and types must be identical
	* ```SPLIT```
	  * takes a single relation and splits them up into more than 1 relations 
* These are used for Diagnostics
	* ```DESCRIBE```
	  * just to describe the schema of a relation
	  * names of the colums and their types
	* ```EXPLAIN```
	  * gives insight on how Pig intends to execute a given query
	* ```ILLUSTRATE```
	  * gives you the step-by-step execution of a sequence of statements
* These are used for User Defined Functions (UDF)
  * Following are used for managing UDF's
    * ```REGISTER```
      * for importing a UDF (which is a jar file)
    * ```DEFINE```
      * is used to assign names to those functions
    * ```IMPORT```
       * is used for importing macros
       * these are reusable piece of codes 
* Some other functions and loaders
  * ```AVG```
    * for calculating average of a column
  * ```CONCAT```
    * for concatening two or more expressions
  * ```COUNT```
    * for counting number of elements in a bag
  * ```MAX```
    * to get the highest value in a column
  * ```MIN```
    * to get the lowest value in a column
  * ```SIZE```
    * to compute number of elements
* Some Storage Classes are
  * ```PigStorage```
    * uses field based data assuming some sort of delimiter on each row 
  * ```TextLoader```
    * just loads up one line of input data
  * ```JsonLoader```
    * for loading JSON data
  * ```AvroStorage```
    * format specifically for serialization and de-serialization
  * ```ParquetLoader```
    * column oriented data format
  * ```OrcStorage```
    * popular for compressed data format
  * ```HBaseStorage```
    * for integrating Pig with HBase which is NoSQL database
* For more details, you can refer to these 
  * [Programming Pig: Dataflow Scripting with Hadoop](https://www.amazon.com/Programming-Pig-Alan-Gates/dp/1449302645)
  * [Pig Latin Basics](https://pig.apache.org/docs/latest/basic.html)
  * [Pig Workshop](https://www.slideshare.net/Sudar/pig-workshop)
  
### Pig Challenge 01
***Find the most popular bad movies***
* The approach will be similar to the [above example](#pig-latin-example)
* We will consider **bad movies** as those with ratings less than **2**
* Since the movies also need to be **popular**, we need to sort them by their *number of ratings*
* We can use ```COUNT``` on ratings to get the popularity sorted
  * the more the count of ratings, the more popular the movie is
* There can be multiple ways to write a Query for this challenge
  * One way could be
```sql
# MostPopularBadMovies
ratings = LOAD '/user/maria_dev/ml-100k/u.data' AS (userID:int, movieID:int, rating:int, ratingTime:int);

metadata = LOAD '/user/maria_dev/ml-100k/u.item' USING PigStorage('|')
	AS (movieID:int, movieTitle:chararray, releaseDate:chararray, videoRelease:chararray, imdbLink:chararray);
	
nameLookup = FOREACH metadata GENERATE movieID, movieTitle;

groupedRatings = GROUP ratings BY movieID;
	
averageRatings = FOREACH groupedRatings GENERATE group AS movieID, AVG(ratings.rating) AS avgRating,
	COUNT(ratings.rating) AS numRatings;

badMovies = FILTER averageRatings BY avgRating < 2.0;

namedBadMovies = JOIN badMovies BY movieID, nameLookup BY movieID;

finalResults = FOREACH namedBadMovies GENERATE nameLookup::movieTitle AS movieName,
	badMovies::avgRating AS avgRating, badMovies::numRatings AS numRatings;

finalResultsSorted = ORDER finalResults BY numRatings DESC;

DUMP finalResultsSorted;
```

## Spark
* Spark is a fast and general engine for large-scale data processing
* Can run programs upto 100x faster than Hadoop MapReduce in memory
  * or 10x faster on disk
* **Directed Acyclic Graph** Engine (DAG) optimizes workflow
* It is currently used by many organizations and is trending
  * Amazon
  * Ebay: log analysis and aggregation
  * NASA JPL: Deep Space Network
  * Groupon
  * TripAdviser
  * Yahoo
    * and many others are using Spark for real massive data
* It's not that hard
  * Can code in Python, Java, or Scala
* It is built around one main concept
  * **the Resilient Distributed Dataset** (RDD)
  
### Scalable
* Spark has a Driver program
  * A script that controls whats going to happen in a job
    * also known as **Spark Context**
* This job goes through Cluster manager
  * Spark can use its own cluster manager
  * Or it can use **YARN**, or even **MESOS**
  * Cluster manager distributes the job in the cluster 
* Now the **Executor units** perform the tasks in parallel
  * Executor units also have **cache**
  * this cache is an important part for speeding up the tasks
    * spark provides a **memory based solution** and doesn't hit HDFS for the file everytime
    * it tries to retain as much as data in RAM

### Components of Spark
* **Spark Streaming**
  * you can input data in real time (as it is being produced)
    * instead of batch processing 
* **Spark SQL**
  * SQL interface for Spark
    * to transform your dataset using SQL queries
* **MLLib**
  * Library of Machine learning and Datamining tools for Spark
* **GraphX**
  * for analyzing Graph theory and its properties (connections, shortest route etc)
    * e.g. Social network graph

### Resilient Distributed Dataset (RDD)
* It's an abstraction of data
  * It makes sure that the job is evenly distributed across the cluster
* It can handle failures in a resilient manner
* It is Resilient and Distributed, but user can just think of it as a data object
* Driver program creates Spark Context
  * its the environment for running RDDs within
  * it creates RDD

#### Creating RDD's
* ```nums = parallelize([1, 2, 3, 4])```
  * that's too small of a dataset, doesn't make sense to distribute to a cluster
* ```sc.textFile("file:///c:/users/frank/gobs-o-text.txt")```
  * or ```s3c://``` for Amazon S3 services
  * ```hdfs://```
* ```hiveCtx = HiveContext(sc); rows = hiveCtx.sql("SELECT name, age FROM users")```
  * can also load data from Hive
* Can also create from
  * Cassandra
  * HBase
  * Elasticsearch
  * JSON, CSV, sequence files

#### Transforming RDD's
* map
  * can call map on the RDD that will apply some function to every input row of your RDD
  * and create a new RDD that is transformed in some way using map
  * one to one relationship between input and output rows
* flatmap
  * when you might need to discard some inputs
  * so the relation is not one to one between input and output rows
* filter
* distinct
* sample
* union, intersection, subtract, cartesian

#### map example
* Taking an RDD and performing some operation on them
  * for eg. squaring them
```python
rdd = sc.parallelize([1,2,3,4])
squaredRDD = rdd.map(lambda x: x*x)
# output
1, 4, 9, 16
```

#### RDD actions
* collect
  * take whatever is in RDD and return a python object to a driver script
* count
  * how many rows are in RDD
* countByValue
  * how many unique rows by values 
* take
  * more like dump
* top
  * more like dump
* reduce
  * combine all the values associated with each key
* In Spark, nothing actually happens in your driver program
  * **until an action is called!**
  * lazy evaluation
  * Spark figures out the fastest way to perform these actions

#### Spark Example
***Find the worst movies in the movielens dataset***
* Worst could be defined as the lowest average ratings
* First thing we will do is create a dictionary for mapping movieID to movieName
  * this information is available in **u.item file**
  * create a function named ```loadMovieNames``` for this operation
* Then load the dataset from the cluster
  * **u.data** has all the movieID's and ratings information
* Since we plan to reduce the ratings per movieID
  * reformat data as ```(movieID, (rating, 1.0))```
  * so that each row with same movieID gets their ratings reduced (summed up)
* Now map movieID's with their average ratings
* Sort the results and display them 
* The python code would be
```python
# LowestRatedMovieSpark.py
from pyspark import SparkConf, SparkContext

# This function just creates a Python "dictionary" we can later
# use to convert movie ID's to movie names while printing out
# the final results.
def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

# Take each line of u.data and convert it to (movieID, (rating, 1.0))
# This way we can then add up all the ratings for each movie, and
# the total number of ratings for each movie (which lets us compute the average)
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))

if __name__ == "__main__":
    # The main script - create our SparkContext
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)

    # Load up our movie ID -> movie name lookup table
    movieNames = loadMovieNames()

    # Load up the raw u.data file
    lines = sc.textFile("hdfs:///user/maria_dev/ml-100k/u.data")

    # Convert to (movieID, (rating, 1.0))
    movieRatings = lines.map(parseInput)

    # Reduce to (movieID, (sumOfRatings, totalRatings))
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: ( movie1[0] + movie2[0], movie1[1] + movie2[1] ) )

    # Map to (rating, averageRating)
    averageRatings = ratingTotalsAndCount.mapValues(lambda totalAndCount : totalAndCount[0] / totalAndCount[1])

    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda x: x[1])

    # Take the top 10 results
    results = sortedMovies.take(10)

    # Print them out:
    for result in results:
        print(movieNames[result[0]], result[1])
```

#### Running the Spark Script
* To run the above example, first login to the cluster
  * Follow the steps from here: [Login Using Putty](#login-using-putty)
* Make python script ```LowestRatedMovieSpark.py``` in the current directory
* You also need to have the movieLens dataset (u.item) in the ml-100k directory
  * If not, use ```mkdir ml-100k```
  * ```cd ml-100k```
  * ```wget http://media.sundog-soft.com/hadoop/ml-100k/u.item```
  * ```cd ..```
* Now run the Spark script using Spark submit, which set's up Spark environment and makes sure that the script is running on your cluster
  * ```spark-submit LowestRatedMovieSpark.py```
* Output will be displayed on the terminal after a few seconds

#### Spark Challenge 01
***Filter out the movies with very few ratings***

```python
# LowestRatedPopularMovieSpark.py

from pyspark import SparkConf, SparkContext

# This function just creates a Python "dictionary" we can later
# use to convert movie ID's to movie names while printing out
# the final results.
def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

# Take each line of u.data and convert it to (movieID, (rating, 1.0))
# This way we can then add up all the ratings for each movie, and
# the total number of ratings for each movie (which lets us compute the average)
def parseInput(line):
    fields = line.split()
    return (int(fields[1]), (float(fields[2]), 1.0))

if __name__ == "__main__":
    # The main script - create our SparkContext
    conf = SparkConf().setAppName("WorstMovies")
    sc = SparkContext(conf = conf)

    # Load up our movie ID -> movie name lookup table
    movieNames = loadMovieNames()

    # Load up the raw u.data file
    lines = sc.textFile("hdfs:///user/maria_dev/ml-100k/u.data")

    # Convert to (movieID, (rating, 1.0))
    movieRatings = lines.map(parseInput)

    # Reduce to (movieID, (sumOfRatings, totalRatings))
    ratingTotalsAndCount = movieRatings.reduceByKey(lambda movie1, movie2: ( movie1[0] + movie2[0], movie1[1] + movie2[1] ) )

    # Filter out movies rated 10 or fewer times
    popularTotalsAndCount = ratingTotalsAndCount.filter(lambda x: x[1][1] > 10)

    # Map to (rating, averageRating)
    averageRatings = popularTotalsAndCount.mapValues(lambda totalAndCount : totalAndCount[0] / totalAndCount[1])

    # Sort by average rating
    sortedMovies = averageRatings.sortBy(lambda x: x[1])

    # Take the top 10 results
    results = sortedMovies.take(10)

    # Print them out:
    for result in results:
        print(movieNames[result[0]], result[1])
```

## Spark SQL
* Spark SQL is used when you're dealing with structured data
* Extends RDD's to DataFrame object
* DataFrames:
  * contain Row objects
  * can run SQL queries
  * have a schema (leading to more efficient storage)
  * read and write to JSON, Hive, parquet
  * communicates with JDBC/ODBC, Tableau
* In **Spark 2.0**, DataFrame is really a DataSet of Row objects
  * Spark 2.0 way is to use DataSets instead of DataFrames when you can

#### Shell Access
* Spark SQL exposes a JDBC/ODBC server
  * if you built Spark with **Hive** support
* Start it with ```sbin/start-thriftserver.sh```
* Listens on port 10000 by default
* Connect using ```bin/beeline -u jdbc:hive2://localhost:10000```
* Now you have a SQL shell to Spark SQL
* You can create new tables, or query existing ones that were cached
  * using ```hiveCtx.cacheTable("tableName")```

#### Spark UDFs
* Spark SQL is extensible
  * you can write your own functions to perform operations inside queries
* Example:
  * Loading a column and performing some operation on it at the same time
    * for example squaring them
```python
from pyspark.sql.types import IntegerType
hiveCtx.registerFunction("square", lambda x: x*x, IntegerType())
df = hiveCtx.sql("SELECT square('someNumericFiled') FROM tableName")
```

#### Spark SQL example
*Find the worst movies in the movielens dataset*
  * **using Spark 2.0**
* Unlike before, we will use ```SparkSession``` instead of ```SparkContext```
  * We will execute SQL queries using this Spark Session object
* We will also use ```Row``` object
  * it will let us create DataFrame of row objects
* We will also import ```functions```
  * it let's us perform some SQL functions (e.g. average) on the data
* ```getOrCreate()``` in Spark Session is used to create a new session
  * or re-run previous Session if it wasn't stopped
* Rest of the things are just easy SQL operations
```python
# LowestRatedMovieDataFrame.py

from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions

def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

def parseInput(line):
    fields = line.split()
    return Row(movieID = int(fields[1]), rating = float(fields[2]))

if __name__ == "__main__":
    # Create a SparkSession (the config bit is only for Windows!)
    spark = SparkSession.builder.appName("PopularMovies").getOrCreate()

    # Load up our movie ID -> name dictionary
    movieNames = loadMovieNames()

    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    # Convert it to a RDD of Row objects with (movieID, rating)
    movies = lines.map(parseInput)
    # Convert that to a DataFrame
    movieDataset = spark.createDataFrame(movies)

    # Compute average rating for each movieID
    averageRatings = movieDataset.groupBy("movieID").avg("rating")

    # Compute count of ratings for each movieID
    counts = movieDataset.groupBy("movieID").count()

    # Join the two together (We now have movieID, avg(rating), and count columns)
    averagesAndCounts = counts.join(averageRatings, "movieID")

    # Pull the top 10 results
    topTen = averagesAndCounts.orderBy("avg(rating)").take(10)

    # Print them out, converting movie ID's to names as we go.
    for movie in topTen:
        print (movieNames[movie[0]], movie[1], movie[2])

    # Stop the session
    spark.stop()
```

#### Running Spark SQL
* Follow the login and data downloading steps from: [Running the Spark Script](#running-the-spark-script)
* We want to use Spark 2.0, type
  * ```export SPARK_MAJOR_VERSION=2```
  * Now the script will run using Spark version 2.0 and we can use SparkSession and other features
* Run the script as:
  * ```spark-submit LowestRatedMovieDataFrame.py```
  
#### Spark SQL Challenge 01
***Filter out the movies with very few ratings***

```python
# LowestRatedPopularMovieDataFrame.py

from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions

def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1]
    return movieNames

def parseInput(line):
    fields = line.split()
    return Row(movieID = int(fields[1]), rating = float(fields[2]))

if __name__ == "__main__":
    # Create a SparkSession (the config bit is only for Windows!)
    spark = SparkSession.builder.appName("PopularMovies").getOrCreate()

    # Load up our movie ID -> name dictionary
    movieNames = loadMovieNames()

    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.data")
    # Convert it to a RDD of Row objects with (movieID, rating)
    movies = lines.map(parseInput)
    # Convert that to a DataFrame
    movieDataset = spark.createDataFrame(movies)

    # Compute average rating for each movieID
    averageRatings = movieDataset.groupBy("movieID").avg("rating")

    # Compute count of ratings for each movieID
    counts = movieDataset.groupBy("movieID").count()

    # Join the two together (We now have movieID, avg(rating), and count columns)
    averagesAndCounts = counts.join(averageRatings, "movieID")

    # Filter movies rated 10 or fewer times
    popularAveragesAndCounts = averagesAndCounts.filter("count > 10")

    # Pull the top 10 results
    topTen = popularAveragesAndCounts.orderBy("avg(rating)").take(10)

    # Print them out, converting movie ID's to names as we go.
    for movie in topTen:
        print (movieNames[movie[0]], movie[1], movie[2])

    # Stop the session
    spark.stop()
```
  
## Using MLLib in Spark 2.0
***Recommend movies to the User according to his past rated movies record***
* Apache Spark MLlib enables building recommendation models from billions of records
  * in just a few lines of Python
* Recommendation algorithms are usually divided into:
  1. **Content-based filtering:**
     * recommending items similar to what users already like
  2. **Collaborative filtering:**
     * recommending items based on what similar users like
       * e.g. recommending video games after someone purchased a game console because other people who bought game consoles also bought video games
* Spark MLlib implements a collaborative filtering algorithm called **Alternating Least Squares** (ALS)
  * available in ```pyspark.ml.recommendation```
* For testing the recommendations, we will create a new user in u.data file
```
0   50    5   881250949
0   172   5   881250949
0   133   1   881250949
```
  * ```userID``` is 0
  * ```movieID``` is 50 
    * which is Star Wars
  * ```rating``` is 5 stars
  * ```timestamp``` can be any value
* We can see that this user likes Sci-Fi movies
  * since he rated 5 stars to movieIDs 50 (Star Wars) and 172 (Empire Strikes Back)
* And hates Drama/Romance genres
  * since he rated 1 star to the movieID 133 (Gone with the Wind)
  
```python
# MovieRecommendationsALS.py

from pyspark.sql import SparkSession
from pyspark.ml.recommendation import ALS
from pyspark.sql import Row
from pyspark.sql.functions import lit

# Load up movie ID -> movie name dictionary
def loadMovieNames():
    movieNames = {}
    with open("ml-100k/u.item") as f:
        for line in f:
            fields = line.split('|')
            movieNames[int(fields[0])] = fields[1].decode('ascii', 'ignore')
    return movieNames

# Convert u.data lines into (userID, movieID, rating) rows
def parseInput(line):
    fields = line.value.split()
    return Row(userID = int(fields[0]), movieID = int(fields[1]), rating = float(fields[2]))


if __name__ == "__main__":
    # Create a SparkSession (the config bit is only for Windows!)
    spark = SparkSession.builder.appName("MovieRecs").getOrCreate()

    # Load up our movie ID -> name dictionary
    movieNames = loadMovieNames()

    # Get the raw data
    lines = spark.read.text("hdfs:///user/maria_dev/ml-100k/u.data").rdd

    # Convert it to a RDD of Row objects with (userID, movieID, rating)
    ratingsRDD = lines.map(parseInput)

    # Convert to a DataFrame and cache it
    ratings = spark.createDataFrame(ratingsRDD).cache()

    # Create an ALS collaborative filtering model from the complete data set
    als = ALS(maxIter=5, regParam=0.01, userCol="userID", itemCol="movieID", ratingCol="rating")
    model = als.fit(ratings)

    # Print out ratings from user 0:
    print("\nRatings for user ID 0:")
    userRatings = ratings.filter("userID = 0")
    for rating in userRatings.collect():
        print movieNames[rating['movieID']], rating['rating']

    print("\nTop 20 recommendations:")
    # Find movies rated more than 100 times
    ratingCounts = ratings.groupBy("movieID").count().filter("count > 100")
    # Construct a "test" dataframe for user 0 with every movie rated more than 100 times
    popularMovies = ratingCounts.select("movieID").withColumn('userID', lit(0))

    # Run our model on that list of popular movies for user ID 0
    recommendations = model.transform(popularMovies)

    # Get the top 20 movies with the highest predicted rating for this user
    topRecommendations = recommendations.sort(recommendations.prediction.desc()).take(20)

    for recommendation in topRecommendations:
        print (movieNames[recommendation['movieID']], recommendation['prediction'])

    spark.stop()
```

* Save this script as ```MovieRecommendationsALS.py```
* Now run the script on Spark 2.0 as defined [here](#running-the-spark-script)

## Hive
* Distributing SQL queries with Hadoop
  * it lets you write standard SQL queries
  * translates SQL queries to MapReduce of TEZ jobs on your cluster
#### Why Hive
* User familiar SQL syntax (HiveQL)
* Interactive
* Scalable - works with big data on a cluster
  * really most appropriate for data warehouse applications
* Easy OLAP queries (Online Analytical Processing)
  * way easier than writing MapReduce in Java
* Highly optimized
* Highly extensible
  * UDF's
  * Thrift Server (talk to Hive from a service)
  * JDBC / ODBC driver
#### Why not Hive
* High latency - not appropriate for OLTP (transaction processing)
  * translates sql commands to MapReduce jobs
  * takes time to produce the output
* Stores data de-normalized
  * not a real relational database
  * just a text file temporarily converted as a table
* SQL is limited in what it can do
  * Pig, Spark allows more complex stuff
* No transactions
* No record-level updates, inserts, deletes

### HiveQL
* Pretty much MySQL with some extensions
* For example: ```views```
  * can store results of a query into a view
    * which subsequent queries can use as a table
* Allows you to specify how structured data is stored and partitioned

#### HiveQL example
***Find movies which are rated by most of the users***
* Goto http://localhost:8080/#/login
  * username: ```maria_dev```
  * password: ```maria_dev```
* Goto **Hive View**
* Upload the dataset by clicking Upload Table
  * Click on settings icon and change ```Field Delimiter``` to TAB(horizontal tab)
    * since our data is tab separated
  * Upload **u.data** from local system
  * change table name as ```ratings```
  * rename first column to ```userID```
  * rename second column to ```movieID```
  * rename third column to ```rating```
  * rename last column to ```timestamp```
  * Now click on **Upload Table** button
* Since we also want to display the names of the movies, we need to import another dataset
  * click on Upload Table button
  * Click on settings icon and change ```Field Delimiter``` to | (pipe character)
    * since our data is pipe char separated
  * select **u.item** from local system
  * change table name as ```names```
  * rename first column to ```movieID```
  * rename second column to ```title```
  * rename last column to ```released```
  * Now click on **Upload Table** button
* Now click on **Query** to start writing the HiveQL interactive query
```mysql
CREATE VIEW topMovieIDs AS
SELECT movieID, count(movieID) AS ratingCount
FROM ratings
GROUP BY movieID
ORDER BY ratingCount DESC;

/* A view allows a query to be saved and treated like a table*/

SELECT n.title, ratingCount
FROM topMovieIDs t JOIN names n ON t.movieID = n.movieID;
```
* Click on Execute button and the results of the query will be displayed
* Click refresh icon on Database Explorer
  * you will see the view ```topMovieIDs```
  * we need to clean up since we're done with the query and got our results
  * to delete this view, type
  ```mysql
  DROP VIEW topMovieIDs;
  ```

#### How Hive Works
* **Schema on Read**
  * Other relational db's use schema on write
    * where you define your db structure before writing into the db
  * Hive maintains a metastore that imparts a structure you define on the unstructured data that is stored on HDFS etc
    * it takes unstructured data, and applies schema to it as it is read
  * In the [HiveQL example](#HiveQL-example), we used Ambari UI to import the data into a table
    * but under the hood, following command was executed
    ```mysql
    CREATE TABLE ratings (
       userID INT,
       movieID INT,
       rating INT,
       time INT)
    ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    STORED AS TEXTFILE;
       
    LOAD DATA LOCAL INPATH '${env:HOME}/ml-100k/u.data'
    OVERWRITE INTO TABLE ratings;
    ```
* **Where is the data**
  * LOAD DATA
    * MOVES data from a distributed filesystem into Hive
  * LOAD DATA LOCAL
    * COPIES data from your local filesystem into Hive
  * Managed vs External tables
    * External means that Hive will not be managing or copying data itself
    * Hive will not take ownership of this data
      * and if you DROP this table, the original data will remain unchanged
    * Used when you want to share the database with other systems as well
    * We can create external table by:
    ```mysql
    CREATE EXTERNAL TABLE IF NOT EXISTS ratings (
       userID  INT,
       movieID INT,
       rating  INT,
       time    INT)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LOCATION '/data/ml-100k/u.data';
    ```
* **Partitioning**
  *  You can store your data in partitioned subdirectories
     * Huge optimization if your queries are only on certain partitions
     ```mysql
     CREATE TABLE customers (
        name    STRING,
        address STRUCT<street:STRING, city:STRING, state:STRING, zop:INT>
     )
     PARTITIONED BY (country STRING);
     ```
  * Hive will partition the data as
    * ```.../customers/country=PK/```
    * ```.../customers/country=AU/```
  * Can be used if you're querying specific to a given partition
    * example. only querying on data where country is Australia
  * You can also used ```structs``` as used in this example
* **Ways to use Hive**
   * Interactive via hive> prompt / Command line interface (CLI)
   * Saved query files
     * hive -f /somepath/queries.hql
   * Through Ambari / Hue
   * Through JDBC/ODBC server
   * Through Thrift service (web clients)
     * But remember, Hive is not suitable for OLTP
   * Via Oozie (scheduler)
   
#### HiveQL Challenge 01
***Find the movie with the highest average rating***
   * Only consider movies with more than 10 ratings count
```mysql
CREATE VIEW IF NOT EXISTS avgRatings AS
SELECT movieID, AVG(rating) AS avgRating, count(movieID) AS ratingCount
FROM ratings
GROUP BY movieID
ORDER BY avgRating DESC;

/* A view allows a query to be saved and treated like a table*/

SELECT n.title, avgRating
FROM avgRatings t JOIN names n ON t.movieID = n.movieID
WHERE ratingCount > 10;
```

## Sqoop
* Command-line interface application for transferring data between relational databases and Hadoop
* Integrating MySQL and Hadoop
* Sqoop can handle Big data
  * kicks off MapReduce jobs to handle importing or exporting your data
  * these mappers and reducers export your data to HDFS
* To import data from MySQL to HDFS
  * First we need to set appropriate permissions so that Sqoop can access the table
  * ```mysql -u root -p```
    * password is ```hadoop```
  * ```GRANT ALL PRIVILEGES ON movielens.* to ''@'localhost';```
    * Now type ```exit``` and import the data from MySQL to HDFS as:
   ```sqoop
   sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver
   	--table movies
   ```
     * it will extract all the data from ```movies``` and store it to HDFS
   * Goto **Files View** > user > maria_dev > movies
     * You can confirm that data from MySQL was stored to HDFS here in **movies directory**
     
* To import data from MySQL directly into Hive
   ```sqoop
   sqoop import --connect jdbc:mysql://localhost/movielens --driver com.mysql.jdbc.Driver
   	--table movies --hive-import
   ```
     * now we can run queries on ```movies``` table through HiveQL
     * To confirm, goto **Hive view** > default database > movies table
     
* To export data from Hive to MySQL
  * remember that the data is in plain text format
  * Hive just structures it for us (Schema on read)
  * Goto **Files View** > apps > hive > warehouse > movies
    * you can open this file and check it is just in plain text format
  * We need to create the table before hand so we could export to it
  * Type ```mysql -u root -p```
    * password is ```hadoop```
  * ```use movielens;```
  * ```CREATE TABLE exported_movies (id INTEGER, title VARCHAR(255), releaseDate DATE);```
    * Now type ```exit``` to go back to terminal
  
   ```sqoop
   sqoop export --connect jdbc:mysql://localhost/movielens -m 1 --driver com.mysql.jdbc.Driver
   	--table exported_movies --export-dir /apps/hive/warehouse/movies
		--input-fields-terminated-by '\0001'
   ```
     * ```-m 1``` means that we will just use 1 MapReduce task (since we have only 1 machine right now)
     * ```--input-fields-terminated-by``` specifies what delimiters are being used for the input fields in the Hive table
        * by default its ASCII value 1
     * ```exported_movies``` table must already exist in MySQL, with columns in expected order
     
   * After this command has finished, go back to mysql terminal
   * Confirm the export was successful by typing
     * ```use movielens;```
     * ```SELECT * FROM exported_movies limit 10;```

#### Importing MovieLens data into a MySQL db
* [Login using Putty](#login-using-putty)
* MySQL comes pre-installed on Horton works sandbox
* Type ```mysql -u root -p```
  * default password is ```hadoop```
* Type ```create database movielens;```
  * to verify, type ```show databases;```
* Type ```exit``` and download the script by typing ```wget http://media.sundog-soft.com/hadoop/movielens.sql```
  * movielens.sql contains mysql script to populate the database
* Log in to mysql again by ```mysql -u root -p```
  * default password is ```hadoop```
* Type ```SET NAMES 'utf8';```
  * some characters cannot be represented in pure ASCII, like **ñ** or **ö**
  * therefore we need to configure MySQL for utf8 encoding
  * MySQL instance is not configured to expect UTF-8 encoding by default
  * the above command does that
* Type ```SET CHARACTER SET utf8;```
  * same explanation as above
* Type ```use movielens;```
  * to use the database we created above for import
* Type ```source movielens.sql;```
  * to import the data using script
  * the data is imported to the database ```movielens``` after this step
* Type ```show tables;```
  * To check the tables in movielens database
* To get few rows from a table, type ```SELECT * FROM tableName LIMIT 10;```
* To view the table columns and its structure, type ```DESCRIBE tableName;```
* ***To find the popular movies using Sqoop***
  ```mysql
  SELECT movies.title, COUNT(ratings.movie_id) AS ratingCount
  	FROM movies INNER JOIN ratings
		ON movies.id = ratings.movie_id
		GROUP BY movies.title
		ORDER BY ratingCount;
  ```

## HBase
* HBase is an open-source, non-relational, scalable database
  * built on top of HDFS, so its also distributed
  * modeled after **Google's Bigtable** and written in Java
* There is no query language for HBase, only CRUD API's
  * Create
  * Read
  * Update
  * Delete
  
#### HBase Architecture
* <p align="center"><img src="https://i.imgur.com/XiUENRT.png"></p>
* This is the high level view of HBase architecture
* It is split up into different Region Servers
  * these aren't geographic regions splits
  * these are ranges of keys
    * just like sharding or range partitioning
  * these can automatically repartition as the data grows
* These Region servers are stored and managed on HDFS
* The web applications or servers communicate with Region Servers directly
* The Master servers are responsible for keeping the track of the actual schema of your data
  * where the data is stored
  * and how it is partitioned
  * it is the mastermind of the HBase cluster that knows where everything is
* Zookeeper is a higly available system that keeps track of who the current Master is
  * if one Master server goes down, Zookeeper will manage it and create a new Master server

#### HBase data model
* Fast access to any given ROW
* A ROW is referenced by a unique KEY
* Each ROW has some small number of COLUMN FAMILIES
* A COLUMN FAMILY may contain arbitrary COLUMNS
* You can have a very large number of COLUMNS in a COLUMN FAMILY
* Each CELL can have many VERSIONS with given timestamps
* Sparse data is OK - missing columns in a row consume no storage
* **Example: One row of a web table from Google's BigTable**
* <p align="center"><img src="https://i.imgur.com/vYEOEjo.png"></p>
* The Key is stored in lexicographic order
* Contents Column Family has only 1 column *Contents*
  * Contents of www.cnn.com are stored in reversed-timestamp order
    * which means it's easy and fast to ask for the latest value
    * but hard to ask for the oldest value
* Anchor column family can have many columns
  * the format of columns is ```Anchor:websiteWhichLinksToWww.Cnn.Com```
  * and the value of this column is the anchor text given by that website

#### Some ways to access HBase
* HBase shell
* Java API
  * Wrappers for Python, Scala, etc
* Connectors for Spark, Hive, Pig
* REST service
* Thrift service
* Avro service

#### Creating a HBase table with Python via REST
* Create a HBase table for movie ratings by user
* Then show we can quickly query it for individual users
* Good example of sparse data
* <p align="center"><img src="https://i.imgur.com/JzD6c2t.png"></p>
* HBase runs on top of HDFS
* We will use a **REST service** to communicate between **Python client** and **HBase**
* First we need to openup the port 8000 so that Python client could communicate with the REST service
* Right click on Horton Works Sandbox and goto Settings
  * Goto Network > Advanced > Port forwarding
  * Click on Add (+) and open port 8000
  * Name: HBase REST
  * Protocol: TCP
  * Host IP: 127.0.0.1
  * Host Port: 8000
  * Guest IP: 
  * Guest Port: 8000
* Now start the REST service through Ambari
  * Goto HBase
  * "Start" from Service Actions
* Now [Login to the VM through Putty](#login-using-putty)
* ```su root```
  * ```/usr/hdp/current/hbase-master/bin/hbase-daemon.sh start rest -p 8000 --infoport 8001```
* Now we need **starbase** which is the REST client for HBase
* On client machine,
  * ```pip install starbase```
* Write the script

```python
# HBaseRESTPython.py

from starbase import Connection

c= Connection("127.0.0.1", "8000")

ratings = c.table('ratings')

if (ratings.exists()):
   print("Dropping existing ratings table\n")
   ratings.drop()

ratings.create('rating')

print("Parsing the ml-199k ratings data...\n")
ratingFile = open("E:/Downloads/ml-100k/ml-100k/u.data", "r")

batch = ratings.batch()

for line in ratingFile:
   (userID, movieID, rating, timestamp) = line.split()
   batch.update(userID, {'rating': {movieID: rating}})
   
ratingFile.close()

print("Committing ratings data to HBase via REST service\n")
batch.commit(finalize=True)

print("Get back ratings for some users...\n")
print("Ratings for user ID 1:\n")
print(ratings.fetch("1"))
print("Ratings for user ID 33:\n")
print(ratings.fetch("33"))
```

* Run the script and it will give the output
  * Movie ratings for a UserID in a row format
* When you are done with the REST service, stop it
  * ```/usr/hdp/current/hbase-master/bin/hbase-daemon.sh stop rest```
  
### Integrating Pig with HBase
* Must create HBase table ahead of time
* Your relation must have a unique key as its first column
  * followed by subsequent columns as you want them saved in HBase
* ```USING``` clause allows you to STORE into an HBase table
* Can work at scale
  * HBase is transactional on rows
* Goto Files View in Ambari
  * user > maria_dev > ml-100k > Upload
  * Upload the u.user file on HDFS
    * contains userID, age, gender, occupation, and zip code 
    * data is pipe delimited (|)
* Login to the [Virtual Machine with Putty](login-using-putty)
* ```hbase shell```
  * ```CREATE 'users', 'userinfo'```
  * ```LIST``` to check that the table is created
  * ```exit```
* Write the script
```pig
# HBase.pig

ratings = LOAD '/user/maria_dev/ml-100l/u.user'
   USING PigStorage('|')
   AS (userID:int, age:int, gender:chararray, occupation:chararray, zip:int);

STORE ratings INTO 'hbase://users'
   USING org.apache.pig.backend.hadoop.hbase.HBaseStorage ('userinfo:age,
   userinfo:gender, userinfo:occupation, userinfo:zip');
```
* Run the script ```pig HBase.pig```
* ```hbase shell```
  * ```list``` to check that the ```users``` table exists
  * ```scan 'users'``` to look for the data inside users table
* Now if you want to delete the table
  * ```disable 'users'```
  * ```drop 'users'```
  * ```list``` to check if users table is deleted
  * ```exit```
* If you are done with HBase
  * goto Services > HBase > Service Actions > Stop

## Cassandra
* A distributed database with no single point of failure
* Unlike HBase, there is no master node at all
  * every node runs exactly the same software and performs the same functions
* Data model is similar to BigTable / HBase
* It's non-relational
  * but has a limited CQL query language as its interface
  
#### Cassandra's Design Choices
* The **CAP Theorem** says you can only have 2 out of 3
  * Consistency, Availability, Partitiion-tolerance
    * and only Partition-tolerance is a requirement with big data
    * so you really only get to choose between Consistency & availability
* Cassandra favors availability over consistency
  * it is "eventually consistent"
  * but you can specify your consistency requirements as part of your request
  * so it's really "tunable consistency"
  
### CQL
* Cassandra's API is CQL
  * which makes it easy to look like existing database drivers to applications
* CQL is like SQL, but with some big limitations!
  * No JOINS
    * your data must be de-normalized
    * so its still non-relational
  * All queries must be on some primary key
    * Secondary indices are supported, but there's performance bottlenecks
* CQLSH can be used on the command line to create tables, etc
* All tables must be in a *keyspace*
  * keyspaces are like databases
  
### Cassandra and Spark
* **DataStax** offers a Spark-Cassandra connector
* Allows you to read and write Cassandra tables as DataFrames
* Is smart about passing queries on those DataFrames down to the appropriate level
* User cases:
  * Use Spark for analytics on data stored in Cassandra
  * Use Spark to transform data and store it into Cassandra for transactional use

### Installing Cassandra on Horton Works Sandbox
* Login to the cluster [with Putty](#login-using-putty)
* ```su root```
  * Password is ```hadoop``` by default
* ```pip install cassandra-driver```
* If you're unable to download with pip, try
  * ```yum update```
  * ```yum install scl-utils```
    * Software collections - gives ability to switch between python versions
  * ```yum install centos-release-scl-rh```
    * CentOS specific component for scl
  * ```yum install python27```
  * ```scl enable python27 bash```
    * switching to python 2.7
    * ```python -V``` to verify
  * We need to set up necessary repositories to pick up Cassandra packages
  * ```cd /etc/yum.repos.d```
  * ```nano datastax.repo```
  ```
  [datastax]
  name = DataStax Repo for Apache Cassandra
  baseurl = http://rpm.datastax.com/community
  enabled = 1
  gpgcheck = 0
  ```
  * Press ```cntrl+O``` and then ```cntrl+X``` to save the file
  * ```yum install dsc30```
    * DataStax Cassandra package installation
* ```pip install cqlsh```
  * CQL is a shell for interacting with Cassandra
* ```service cassandra start```
  * to start the cassandra service
* ```cqlsh```
  * if you get any version mismatch errors, give the specific supported version while giving this command
    * ```cqlsh --cqlversion="3.4.0"```
* To create a table, we first need to create Keyspace
  * keyspace is like a database in SQL
  * ```CREATE KEYSPACE movielens WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'} AND durable_writes = true;```
* ```USE movielens;```
* ```CREATE TABLE users (user_id int, age int, gender text, occupation text, zip text, PRIMARY KEY (user_id));```
* ```DESCRIBE TABLE users;```
  * to check the newly created table
* ```exit``` to go back to terminal

#### Populating the Cassandra DB with Spark
* Write the script as *CassandraSpark.py*
```python
# CassandraSpark.py

from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions

def parseInput(line):
    fields = line.split('|')
    return Row(user_id = int(fields[0]), age = int(fields[1]), gender = fields[2], occupation = fields[3], zip = fields[4])

if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("CassandraIntegration").config("spark.cassandra.connection.host", "127.0.0.1").getOrCreate()

    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.user")
    # Convert it to a RDD of Row objects with (userID, age, gender, occupation, zip)
    users = lines.map(parseInput)
    # Convert that to a DataFrame
    usersDataset = spark.createDataFrame(users)

    # Write it into Cassandra
    usersDataset.write\
        .format("org.apache.spark.sql.cassandra")\
        .mode('append')\
        .options(table="users", keyspace="movielens")\
        .save()

    # Read it back from Cassandra into a new Dataframe
    readUsers = spark.read\
    .format("org.apache.spark.sql.cassandra")\
    .options(table="users", keyspace="movielens")\
    .load()

    readUsers.createOrReplaceTempView("users")

    sqlDF = spark.sql("SELECT * FROM users WHERE age < 20")
    sqlDF.show()

    # Stop the session
    spark.stop()
```

  * Make sure you have **u.user** file in user > maria_dev > ml-100k directory
    * if not, [login with Ambari](#login-using-ambari) and goto files view to upload that file from [movielens dataset](http://files.grouplens.org/datasets/movielens/ml-100k.zip)
  * The above script reads plain text from u.user file and then parses them into ```Row``` object
    * ```Row``` objects are needed by the Spark DataFrames
  * These ```Row``` objects are then converted into Spark supported ```DataFrames```
  * These dataframe objects can now be written to the Cassandra DB
  * We can also perform queries on the dataset
* To run the script, ```export SPARK_MAJOR_VERSION=2```
* ```spark-submit --packages datastax:spark-cassandra-connector:2.0.0-M2-s_2.11 CassandraSpark.py```
  * it means that it is a Cassandra connector for
    * Scala version 2.11
    * Spark version 2.0.0
  * might be different if you're using a newer version of HortonWorks Sandbox
* To check whether the data was written into Cassandra DB or not
  * ```cqlsh --cqlversion="3.4.0"```
  * ```USE movielens;```
  * ```SELECT * FROM users LIMIT 10;```
  * ```exit```
* ```service cassandra stop```
  * to stop the service when you're done experimenting with Cassandra
  
## MongoDB
* Managing HuMONGOus data
* MongoDB is a free and open-source cross-platform document-oriented database program
* Classified as a NoSQL database program
  * MongoDB uses JSON-like documents with schema
* Example:
```json
{
  "_id": ObjectID("7b2js62k09bae5ea6027sdl271c3"),
  "title": "A blog post about MongoDB",
  "content": "This is a blog post about MongoDB",
  "comments": [
     {
        "name": "Morty",
	"email": "Morty@ricknmorty.com",
	"content": "This is the best article ever written!",
	"rating": 1
     }
  ]
}
```

#### No real schema is enforced
* You can have different fields in every document if you want to
* No single "key" as in other databases
  * But you can create indices on any fields you want
    * or even combinations of fields
  * If you want to "shard", then you must do so on some index
* Results in a lot of flexibility
  * But with great power comes great responsibility

#### MongoDB terminology
* Databases
* Collections
  * just like tables
* Documents
  * just like rows
  
#### Replication Sets
* Single Master
* Maintains backup copies of your database instance
  * Secondaries can elect a new primary within seconds if your primary goes down
  * But make sure your operation log is long enough to give you time to recover the primary when it comes back
  * Applications talk with Primary
  
#### Replica Set Quirks
* A majority of the servers in your set must agree on the primary
  * Even numbers of servers (like 2) don't work well
* Don't want to spend money on 3 servers?
  * You can set up an 'arbiter' node
    * But only one
* Apps must know about enough servers in the replica set to be able to reach one to learn who's primary
* Replicas only address durability, not your ability to scale
  * Well, unless you can take advantage of reading from secondaries
    * which generally isn't recommended
  * And your DB will still go into read-only mode for a bit while a new primary is elected
* Delayed secondaries can be set up as insurance against people doing dumb things

### Sharding
* For scaling out data on more than 1 server with MongoDB, we do Sharding
* Ranges of some indexed value you specify are assigned to different replica sets
  * each replica set is responsible for some range of values on some indexed value in the DB
* In order to do sharding, we need to set up index on some unique value in our collection
* On the application server, you set up ```mongos```
* ```mongos``` talks with 3 config servers running which know about the partitioning information
  * this information is used by mongos to know which replica set to talk with for getting the data
* mongos also run a balancer in the background
  * After some time if it finds out that the data is not evenly partitioned
  * it can rebalance the values across the replica sets in real time
* <p align="center"><img src="https://i.imgur.com/g3Jlnlr.png"></p>

#### Sharding Quirks
* Auto-sharding sometimes doesn't work
  * Split storms, mongos processes restarted too often
* You must have 3 config servers
  * And if any one goes down, your DB is down
  * This is on top of the single-master design of replica sets
* MongoDB's loose document model can be at odds with effective sharding

#### Neat things about MongoDB
* It's not just a NoSQL database
  * very flexible document model
* Shell is a full JavaScript interpreter
  * can run JS functions across your entire MongoDB
* Supports many indices
  * but only one can be used for sharding
  * more than 2-3 are still discouraged
  * Full-text indices for text searches
  * Spatial indices
    * Spatial indices are used by spatial databases (databases which store information related to objects in space)
    * Conventional index types do not efficiently handle spatial queries
    * such as how far two points differ, or whether points fall within a spatial area of interest
* Built-in aggregation capabilities, MapReduce, GridFS
  * for some applications you might not need Hadoop at all
  * But MongoDB still integrates with Hadoop, Spark, and most languages
* A SQL connector is available
  * but MongoDB still isn't designed for joins & normalized data really
 
#### Using Mongo with Spark example
* [Login using Putty](#login-using-putty)
* ```su root```
* ```cd /var/lib/ambari-server/resources/stacks```
* ```cd HDP```
  * now ```ls``` and check switch to the directory version of your Hadoop
* if the Hadoop version is 2.5
  * ```cd 2.5```
* ```cd services```
* Now we will add MongoDB to the services list
  * there is a mongo-ambari connector available on github
* ```git clone https://github.com/nikunjness/mongo-ambari.git```
* Now restart the ambari services for the changes to be effective
  * ```sudo service ambari restart```
* Goto http://127.0.0.1:8080
  * sign in with ```admin``` credentials
* Click on **Actions** button > Add service
  * Select **MongoDB** from the list
  * Click on Next
  * Just accept the default values and keep clicking Next
  * If you get warnings, click on **Proceed anyway**
    * warnings are due to the large number of services running on a single virtual machine
  * Finally click on Deploy
* Now make sure you have **u.user** file in user/maria_dev/ml-100k/ directory
* On putty client, type ```pip install pymongo```
* ```exit``` to switch back to normal user
  * ```cd ~``` to go back to the home directory
* Write the script into ```MongoSpark.py``` file
```python
# MongoSpark.py

from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql import functions

def parseInput(line):
    fields = line.split('|')
    return Row(user_id = int(fields[0]), age = int(fields[1]), gender = fields[2], occupation = fields[3], zip = fields[4])

if __name__ == "__main__":
    # Create a SparkSession
    spark = SparkSession.builder.appName("MongoDBIntegration").getOrCreate()

    # Get the raw data
    lines = spark.sparkContext.textFile("hdfs:///user/maria_dev/ml-100k/u.user")
    # Convert it to a RDD of Row objects with (userID, age, gender, occupation, zip)
    users = lines.map(parseInput)
    # Convert that to a DataFrame
    usersDataset = spark.createDataFrame(users)

    # Write it into MongoDB
    # database is movielens
    # collection is users
    usersDataset.write\
        .format("com.mongodb.spark.sql.DefaultSource")\
        .option("uri","mongodb://127.0.0.1/movielens.users")\
        .mode('append')\
        .save()

    # Read it back from MongoDB into a new Dataframe
    readUsers = spark.read\
    .format("com.mongodb.spark.sql.DefaultSource")\
    .option("uri","mongodb://127.0.0.1/movielens.users")\
    .load()

    readUsers.createOrReplaceTempView("users")

    sqlDF = spark.sql("SELECT * FROM users WHERE age < 20")
    sqlDF.show()

    # Stop the session
    spark.stop()
```
  
* ```export SPARK_MAJOR_VERSION=2```
  * since we want to use Spark 2.0
* ```spark-submit --packages org.mongodb.spark:mongo-spark-connector_2.11:2.0.0 MongoSpark.py```
  * it means that it is a MongoDB connector for
    * Scala version 2.11
    * Spark version 2.0.0
  * might be different if you're using a newer version of HortonWorks Sandbox
  
#### Mongo Examples
* [Login to the shell using Putty](#login-using-putty)
* type ```mongo```
  * ```use movielens``` 
* To find the information about user ID 100, type
  * ```db.users.find( {user_id: 100 } )```
* To know what's going on behind a command, use explain function
  * ```db.users.explain().find( {user_id: 100} )```
    * you will see inside *winningPlan* that it's just scanning collections to find the user ID
    * It just scans all the collections to find the particular user ID which is inefficient
    * We should do indexing on the user ID to make queries faster
* To create index on the user ID
  * ```db.users.createIndex ( {user_id: 1} )```
    * this create an index on user_id in descending order
  * to verify that index was created
  * ```db.users.explain().find( {user_id: 100} )```
    * check the *winningPlan* 
    * you will that now it is doing **IXSCAN** - index scan
    * thus the queries will be alot more efficient now
