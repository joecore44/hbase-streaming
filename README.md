
# ♾ HBASE STREAMING

Everything Below is for CDH3u3 on CentOS 6 but you can figure it out with any other stack.




## INITIAL SETUP

Clone The Project

```bash
  git clone https://github.com/joecore44/hbase-streaming.git
  cd hbase-streaming
  Place hadoop-hbase-streaming.jar in /usr/local/hadoop-hbase-streaming.jar
  Add to : /etc/hadoop-0.20/conf/hadoop-env.sh
  export HADOOP_CLASSPATH="/usr/local/hadoop-hbase-streaming.jar:$HADOOP_CLASSPATH"
  export HADOOP_CLASSPATH="/usr/lib/hbase/lib/guava-r06.jar:$HADOOP_CLASSPATH"
  export HADOOP_CLASSPATH="/usr/lib/hbase/hbase-0.90.4-cdh3u3.jar:$HADOOP_CLASSPATH"
  export HADOOP_CLASSPATH="/usr/lib/zookeeper/zookeeper-3.3.4-cdh3u3.jar:$HADOOP_CLASSPATH" 
```

## LOAD DATA

Set Up Output Table with Columns:

```bash
 create 'outputtable', {NAME=>'cf1'}, {NAME=>'cf2'}
```

Create Reducer With (tab-delimited) Output:

```bash
 put	<rowid>	<cf>:<qualifier>	<value>
```

Run Map Reduce With OutputFormat Set To: org.childtv.hadoop.hbase

An Example of the Reducer (tab-delimited): 

```bash
put	r1	cf1:test	Value1
put	r1	cf2:test	Value2
put	r2	cf1:test	Value3 
```

Then Run the Hadoop Streaming API With Output: org.childtv.hadoop.hbase.mapred.ListTableOutputFormat and the Job Configuration Parameter reduce.output.table=outputtable 

```bash
hadoop jar  /usr/lib/hadoop-0.20/contrib/streaming/hadoop-streaming-0.20.2-cdh3u3.jar \
	-input source_input -output dummy_output \
	-mapper /bin/cat \
	-outputformat org.childtv.hadoop.hbase.mapred.ListTableOutputFormat \
	-jobconf reduce.output.table=outputtable 
```

## EXTRACT DATA

To Read From HBASE Create an Empty Directory

```bash
mkdir starter_input
```

Select Desired InputFormat. Two Exist : JSON: org.childtv.hadoop.hbase.mapred.JSONTableInputFormat Tabular Values: org.childtv.hadoop.hbase.mapred.ListTableInputFormat

Select The Input Column That You Want Using Job Config Param map.input.Columns

JSON Format Ex:

```bash
r1	{"c_2:test":{"timestamp":"1333428648468","value":"Value1"},"c_1:test":{"timestamp":"1333428678724","value":"Value2"}} 
r2	{"c_2:test":{"timestamp":"1333428656033","value":"Value3"},"c_1:test":{"timestamp":"1333428660721","value":"Value4"}} 
```

To Run The Job on an HBase Table Called hbase_table With Columns Called c_1 and C02 Run:

```bash
hadoop jar /usr/lib/hadoop-0.20/contrib/streaming/hadoop-streaming-0.20.2-cdh3u3.jar \
	-input starter_input -inputformat org.childtv.hadoop.hbase.mapred.JSONTableInputFormat \
	-mapper /bin/cat \
	-jobconf map.input.table=hbase_table -jobconf "map.input.columns=c_1 c_2" \
	-output hbase_output 
```

This Will Generate a File in hbase_output/part-00000 With Your JSON
