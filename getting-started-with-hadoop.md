# Getting Started with Hadoop


## Big Data!
+ **"More data usually beats better algorithms"**
+ Data storage & analysis challenge
	+ Storage: data replication in case of failure
	+ Analysis: MapReduce programming model

## Parallel programming
+ serial vs. parallel
+ program processing broke into parts, each of which can be executed concurrently (different CPUs/cores)
+ **MASTER/SLAVE** pattern

## MapReduce
+ Programming model for data processing
+ **Abstraction** that allows engineers to perform simple computations while hiding the details of **parallelization**, **data distribution**, **load balancing** and **fault tolerance** in a library
+ Developed with Google to process large amounts of raw data
+ [Dean, Jeff and Ghemawat, Sanjay. **MapReduce: Simplified Data Processing on Large Clusters**](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/mapreduce-osdi04.pdf)

## Programming Model
Map & Reduce

	map		(k1, v1)		-> list(k2, v2)
	reduce	(k2, list(v2))	-> list(v2)

Word Count example in pseudo-code:
	
	map(String key, String value):
		// key: document name
		// value: document contents
		for each word w in value:
			EmitIntermediate(w, "1");
	
	reduce(String key, Iterator values):
		// key: a word
		// values: a list of counts
		int result = 0;
		for each v in values:
			result += ParseInt(v);
		Emit(AsString(result));

## Execution
![image](http://www.dbthink.com/wp-content/uploads/2010/06//map_reduce_execution_overview.jpg)

## Word Count Execution
![image](http://www.confusedcoders.com/wp-content/uploads/2012/08/wordcount.png)

## About the sorting
When a reduce worker has read all intermediate data, it sorts it by the intermediate keys so that all occurrences of the same key are grouped together. The sorting is needed because typically many different keys map to the same reduce task. If the amount of intermediate data is too large to fit in memory, an external sort is used.

## Hadoop
+ Apache Hadoop
+ **Reliable shared storage** and **analysis system**
	- Storage: Hadoop Distributed File System(HDFS)
	- Analysis: MapReduce
	
## MapReduce Execution in Hadoop
+ A single reduce task
+ Multiple reduce tasks
+ No reduce task

## HDFS
+ Hadoop Distributed File System
+ Distributed file system (network file system)

## Hadoop Installation
### Java
Sun Java on Ubuntu linux, Hadoop not compatible with open SDK, to install sun java on ubuntu linux, refer to [oab-java6](https://github.com/flexiondotorg/oab-java6)

### linux
linux is the only supported production environment

	$ tar xzf hadoop-x.y.z.tar.gz
	$ export JAVA_HOME
	$ export HADOOP_INSTALL
	$ export PATH=$PATH:$HADOOP_INSTALL/bin
	$ hadoop version
	
### mac
only for development
	
	brew install hadoop
	

## Hadoop Configuration

### Standalone (local) mode
+ no daemons
+ single JVM
+ development purpose

### Pseudo-distributed mode
+ local machine
+ simulating a cluster on a small scale

### Fully distrubted mode
+ cluster of machines

### Hadoop configuration files
 /usr/local/Cellar/hadoop/1.0.3/libexec/conf/
 
+ hadoop-env.sh
+ core-site.xml
+ hdfs-site.xml
+ mapred-site.xml

## Word Count with Hadoop

[Mapper Interface](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapreduce/Mapper.html)

	public class Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>
	
[Reducer Interface](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapreduce/Reducer.html)
	
	public class Reducer<KEYIN, VALUEIN, KEYOUT, VALUEOUT>
	
[Mapper.Content](http://hadoop.apache.org/docs/r0.20.2/api/org/apache/hadoop/mapreduce/Mapper.Context.html)
	
	public class Mapper.Context extends MapContext<KEYIN, VALUEIN, KEYOUT, VALUEOUT>

### Java Code
[WordCount.java](WordCount.java)

[Java Generics](http://en.wikipedia.org/wiki/Generics_in_Java)

[Java nested classes](http://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)

Hadoop provides its own set of basic types optimized for network serialization

+ LongWritable: Java Long
+ Text: Java String
+ IntWritable: Java Integer

### Java Code
[Reducer](http://hadoop.apache.org/docs/r0.20.1/api/org/apache/hadoop/mapreduce/Reducer.html)

+ **Shuffle**: The reducer copies the sorted output from each Mapper using HTTP across the network
+ **Sort**: The framework merge sorts Reducer inputs by keys. The shuffle and sort phases occur simultaneously i.e. while outputs are being fetched they are merged.
+ **Reduce**: Reducer.reduce() method called for each <key, (collection of values)>; The output of the Reducer is not re-sorted.


## Test Run (standalone mode)
input directory: input

output directory: output
	
	$ javac -classpath /path/to/hadoop-x.y.z/hadoop-core-x.y.z.jar WordCount.java
	$ jar cvf WordCount.jar *.class
	$ hadoop jar WordCount.jar WordCount wordcount/input wordcount/output
	
### Pseuco-Distributed Mode

Setting passwordless ssh
	
	ssh-keygen -t rsa -P ""
	cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
	ssh localhost

core-site.xml

	<?xml version="1.0"?>
	<!-- core-site.xml -->
	<configuration>		<property>			<name>fs.default.name</name>			<value>hdfs://localhost/</value>		</property>
	</configuration>

hdfs-site.xml
	<?xml version="1.0"?> 	<!-- hdfs-site.xml -->	<configuration>		<property>			<name>dfs.replication</name>			<value>1</value>		</property>
	</configuration>

mapred-site.xml
	<?xml version="1.0"?>	<!-- mapred-site.xml -->	<configuration>		<property>			<name>mapred.job.tracker</name>			<value>localhost:8021</value>		</property>
	</configuration>
	
Execution

	$ clear /tmp/hadoop* (clear datanode)
	
	$ hadoop namenode -format
	
	
	$ start-all.sh
	$ jps
	$ ps ax | grep hadoop | wc -l

	
NameNode - http://localhost:50070

JobTracker - http://localhost:50030

	$ hadoop dfs -mkdir inputwords
	$ hadoop dfs -put input inputwords

	$ hadoop jar WordCount.jar WordCount inputwords outputwords
	$ hadoopo fs -cat wordcount/output/part-r-0000
	$ stop-all.sh