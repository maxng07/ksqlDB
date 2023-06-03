# ksqlDB

This is to document the learnings I have on ksqlDB. Version installed 28.2. I installed kafka and ksqlDB on the same 4 vCPU, 16G RAM VM.

CLI v0.28.2, Server v0.28.2

OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
                  
                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =        The Database purpose-built       =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2022 Confluent Inc.

CLI v0.28.2, Server v0.28.2 located at http://localhost:8088
Server Status: RUNNING

## Configuring ksqlDB on topic with delimiter format
1. create stream botnet (Date VARCHAR, Time VARCHAR, Method VARCHAR, URI VARCHAR,ClientIP VARCHAR, `User-agent` VARCHAR) WITH (kafka_topic='webserver', key_format='delimited', value_format='delimited', VALUE_DELIMITER='SPACE');

2. Once the stream is created, you can query the data using SQL select. ksql does not support ORDER BY 

select * from botnet;
Display all the logs from the topic

select count(*) from botnet where `User-agent` LIKE '%python%' emit changes;

select * from botnet where `User-agent` NOT LIKE '%Mozilla%';

select count(*) as count, `User-agent`, CLIENTIP, METHOD, URI  from botnet GROUP BY `User-agent`, CLIENTIP, METHOD, URI  emit changes;
```
+--------------------+--------------------+--------------------+--------------------+--------------------+
|COUNT               |User-agent          |CLIENTIP            |METHOD              |URI                 |
+--------------------+--------------------+--------------------+--------------------+--------------------+
|1                   |Mozilla/5.0         |54.36.148.43        |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |51.222.253.3        |GET                 |/w3.css             |
|1                   |Mozilla/5.0         |167.248.133.125     |GET                 |/                   |
|1                   |Mozilla/5.0         |167.248.133.125     |GET                 |/favicon.ico        |
|1                   |Mozilla/5.0         |54.36.148.244       |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |54.36.148.240       |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |54.36.148.111       |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |66.249.66.78        |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |54.36.149.100       |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |51.222.253.19       |GET                 |/                   |
|1                   |Mozilla/5.0         |87.236.176.28       |GET                 |/                   |
|1                   |Mozilla/5.0         |167.94.138.50       |GET                 |/                   |
|1                   |Mozilla/5.0         |167.94.138.50       |GET                 |/favicon.ico        |
|1                   |Mozilla/5.0         |66.249.66.38        |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |51.222.253.5        |GET                 |/w3.css             |
|1                   |Mozilla/5.0         |66.249.66.68        |GET                 |/robots.txt         |
|1                   |Mozilla/5.0         |66.249.66.11        |GET                 |/                   |
|1                   |Mozilla/5.0         |66.249.66.9         |GET                 |/                   |
|2                   |Mozilla/5.0         |167.94.146.59       |GET                 |/                   |
|2                   |Mozilla/5.0         |167.94.146.59       |GET                 |/favicon.ico        |
```
With delimiter file with SPACE, you will have to define all the columns. With JSON, you can select which keys you like to be part of the stream to be included, offering greater flexibility of not importing all data. With Delimiter format log, if you space as the "delimit" format, care should be taken as any space between description can be mistook by ksql. I clean up my logs with sed and awk to have meaningful analysis. ksqlDB only works on structure data.



##### Some kafka tricks #####
Deleting messages from kafka topics
kafka-delete-records.sh --bootstrap-server 192.168.0.100:9092  --offset-json-file delete-records.json

Dumping records from the start
kafka-console-consumer.sh --bootstrap-server 192.168.0.100:9092 --topic webserver --from-beginning

Ingesting records
kafka-console-producer.sh --bootstrap-server 192.168.0.100:9092 --topic webserver </yyy/xxx.log

### To be Work On
Will include create stream using json format with struct and array.
