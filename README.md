# Sparkify shifts from Data Warehouse to Data Lake

Sparkify, a startup with a new streaming app, has recently seen an increase in its customer base and song database system. After discussions with an IT consulting firm, the company has decided to move not only its operations to the cloud but also its data warehouse to a data lake. Sparkify hopes this will not only reduce costs, but also allow it to scale flexibly to meet its business needs.


### Table of Contents

- [Project Summary](#project)
- [Spark and EMR](#spark)
- [How to run the Python Scripts](#python)
- [Sparkify Data Files](#files)
- [Design of the Database Schema and ETL Pipeline](#pipeline)
- [Querying the Data](#query)


<a name="project"/>

## Project Summary


After a steady growth of its user base, Sparkify decided to move its data warehouse to a data lake. The company chose AWS as its cloud provider and currently uses S3 as its central repository to store all structured and unstructured data. Currently, all files are stored in JSON format. To extract, load and transform the data, Sparkify will use Spark as the ELT engine for its data lake. Once all the data is collected, processed and loaded back into S3, the analytics team will be able to access it to gain valuable insights into how users are using and consuming the service. 


<a name="spark"/>

## Spark and EMR

Spark:

Apache Spark is an open source data processing engine that uses simple programming constructs to quickly perform processing tasks on very large datasets across multiple computer clusters. Apache Spark supports various programming languages such as R, Python, Java, and Scala.

EMR:

EMR stands for Elastic MapReduce and is a managed cluster platform that facilitates the execution of Big Data frameworks such as Apache Hadoop and Apache Spark on AWS to process and analyze data that is more diverse and arriving in ever-increasing volumes and at higher velocities.



<a name="python"/>

## How to run the Python scripts

In order to be able to run the provided python scripts, the following criteria needs to be fulfilled:
- Python must be installed in version 3.6.3 or higher.
- It is highly recommended to install the necessary Python libraries in a virtual environment.
- Access to an AWS account with permission to access S3 buckets and deploy a Redshift cluster.
- Save all required string connections data in the *dwh.cfg* file

In this case, we will deploy our Spark job using only two files: a configuration file and a Python script etl.py. These are

1. 
2. the *etl.py* script: once the tables are created, we proceed to extract, transform and load the data into our database. This script acts as an orchestration engine between the raw data and our Redshift database. It relies heavily on the queries we develop for the sql_queries.py script. Once this script is executed, the Redshift database is populated with clean data.


<a name="files"/>

## Sparkify Data Files

The files used in this repository are all stored in two **S3 buckets**. These buckets contain the song dataset and the log dataset, both in json format.
```
Song data: s3://<udacity-bucket>/song_data

Log data: s3://<udacity-bucket>/log_data
```

**The song dataset**

The song dataset is a subset of the [MillionDataSet](http://millionsongdataset.com/). These files contain real metadata about songs and artists. As mentioned earlier, it is a subset that contains only the first three letters of the track ID of each song; hence the partitioning.

The song files have the following structure:
```
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```

Below we can see what attributes make up a song file:
```
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
```



**The log Dataset**

Unlike the song files, which are real data, the log files are simulated files. They were created using an external tool ([event simulator](https://github.com/Interana/eventsim)). These files are divided by month; however, only generated data for the month of November was used.

The log file have the following structure:
```
log_data/2018/11/2018-11-12-events.json
log_data/2018/11/2018-11-13-events.json
```

**Output Datasets**

The output files (dimensions and fact tables for our model) are also stored in an S3 bucket. The files here can be used later for further processing and conversion.
Our output bucket has the following structure:
```
Output data: s3://<my-bucket>/<dim name>/<table name.parquet>/
```


<a name="pipeline"/>

## Design of the database schema and ETL pipeline

**Design of the database schema**

This database was developed to contain two staging tables, four dimension tables and one fact table. The data copied from Sparkify S3 buckets where uploaded into our two staging tables. These tables where used to derived the five following tables. The dimension tables were created by grouping related attributes about the numerical values in the fact table, under one dimension; this avoided duplicating data when it was not necessary. As a result, we have the following dimension tables:

- Songs
- Artists
- Users
- Time

The Songplays fact table was created to store partially denormalized data for analytical purposes. While the granularity of dimension tables is theoretically much higher, the fact table takes data with a finer grain.

**ETL Pipeline**

We wrote our ETL pipeline using Python. In doing so, we made extensive use of the **pyscopg2** library. This library allowed us to connect to our Redshit database, create a cursor to execute commands, and run all of our Python scripts from one place. To copy all the json files, perform the necessary transformations, and load the final data into our database, we created three Python scripts. Detailed explanations of the scripts and how they work can be found above in the section: **Execution of the Python Scripts**.


<a name="query"/>

## Querying the Data
