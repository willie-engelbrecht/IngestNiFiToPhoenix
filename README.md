# Introduction
Apache Phoenix enables OLTP and operational analytics in Hadoop for low latency applications by combining the best of both worlds:

* the power of standard SQL and JDBC APIs with full ACID transaction capabilities and
* the flexibility of late-bound, schema-on-read capabilities from the NoSQL world by leveraging HBase as its backing store

Apache Phoenix is fully integrated with other Hadoop products such as Spark, Hive, Pig, Flume, and Map Reduce.

[Read more about the Apache Phoenix project](https://phoenix.apache.org/)

The demo below will outline how you can integrate HDF (NiFi) with Phoenix to source data and push to Phoenix for BI Analysis with an ODBC tool (Excel). Any tool that is capable of using an ODBC connection will be able to source this data from Phoenix. 

## Setup
Create your table in Phoenix, by running the command phoenix-sqlline on your terminal:
```
[root@hdp3 ~]# phoenix-sqlline
Setting property: [incremental, false]
Setting property: [isolation, TRANSACTION_READ_COMMITTED]
issuing: !connect jdbc:phoenix: none none org.apache.phoenix.jdbc.PhoenixDriver
Connecting to jdbc:phoenix:
Connected to: Phoenix (version 5.0)
Driver: PhoenixEmbeddedDriver (version 5.0)
Autocommit status: true
Transaction isolation: TRANSACTION_READ_COMMITTED
Building list of tables and columns for tab-completion (set fastconnect to true to skip)...
149/149 (100%) Done
Done
sqlline version 1.2.0

0: jdbc:phoenix:> !sql create table employees (emp_no integer primary key, birth_date varchar, first_name varchar, last_name varchar, gender varchar, hire_date varchar, random_nr integer);
No rows affected (0.819 seconds)
```

You can do the same in Hive3, for the streaming portion:
```
[root@hdp3 ~]$ hive
Connecting to jdbc:hive2://hdp3.1.0-0.home.local:2181/default;password=hive;serviceDiscoveryMode=zooKeeper;user=hive;zooKeeperNamespace=hiveserver2
19/04/23 15:54:31 [main]: INFO jdbc.HiveConnection: Connected to hdp3.1.0-0.home.local:10000
Connected to: Apache Hive (version 3.1.0.3.1.0.0-78)
Driver: Hive JDBC (version 3.1.0.3.1.0.0-78)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.0.3.1.0.0-78 by Apache Hive

0: jdbc:hive2://hdp3.1.0-0.home.local:2181/de> create table employees (emp_no int, birth_date string, first_name string, last_name string, gender string, hire_date string);
INFO  : OK
No rows affected (0.306 seconds)
```

Also create your Kafka topic, using the correct Hostname/IP address of your machine:
```
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --topic topic_data --replication-factor 1 --partitions 2 --zookeeper localhost:2181
```

## Import the NiFi flow
Import the example NiFi flow (demo.xml) into your instance of NiFi:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/UploadTemplate.JPG "Load the template - step 1")

Load select the file on your computer and upload:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/UploadTemplate2.JPG "Upload the template - step 2")

Click on the Template button to load the example to your NiFi instance:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/ClickTemplateButton.JPG "Load the sample flow")

And then load a copy to your canvas:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/LoadTemplateToCanvas.JPG "Load template to the canvas")

Once Imported, update the three Kafka processors, and set the correct Hostname/IP of your Kafka broker:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/KafkaBrokerIP.JPG "Update Kafka Broker Hostname/IP")

Double click on PutSQL, and update the Hostname/IP of your Phoenix server in the DBCPConnectionPool:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/DBCPConnectionPool.JPG "Update DBCPConnection pool")

Also update the Hostname/IP of your HiveMetastore URI in the PutHive3Streaming processor:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/HiveMetaStore.JPG "Update Hive Metastore URI Hostname/IP")

You can now start your flows in the green box, and see the data being push to the Kafka topic:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/GenerateData.JPG "Start your data generation to Kafka")

Start the ingest from Kafka to Phoenix, and Kafka to Hive3Streaming:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/PushToPhoenix.JPG "Push to Phoenix")

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/PushToHive3.JPG "Push to Hive3")

While it's running, you can go back to the commandline and test the row count:
```
sqlline version 1.2.0
0: jdbc:phoenix:> select count(*) from employees;
+-----------+
| COUNT(1)  |
+-----------+
| 30        |
+-----------+
1 row selected (0.086 seconds)
```

Using ODBC, you can use a tool like Excel to load the data for further analyses:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/ImportFromODBC.JPG "Excel - Import from ODBC")

Pick wich DSN to use: 

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/PickYourDSN.JPG "Excel - Pick your DSN")

Pick your table in the Phoenix ODBC connection:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/PickYourTable.JPG "Excel - Pick your Phoenix table")

Now you have imported your data:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/IngestedData.JPG "Excel - View imported data")

You can do the same from Zeppelin, by querying %jdbc(Phoenix), pulling and visualising the data:

Example 1:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/Zeppelin-1.JPG "Zeppelin - Example 1")

Example 2:

![alt text](https://github.com/willie-engelbrecht/IngestNiFiToPhoenix/blob/master/Zeppelin-2.JPG "Zeppelin - Example 2")

