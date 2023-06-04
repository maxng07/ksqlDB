# ksqlDB

This is to document the learnings I have on ksqlDB. Version installed 28.2. I installed kafka and ksqlDB on the same 4 vCPU, 8G RAM VM. This does not cover installations, a lot of guide can be found online in confluent website for downloading and installing ksqlDB.

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

Something to note, there are reserved key words in ksql that you cannot used for columns. A good example is "-", if you have that and need to use it in your table or stream, you can use a back quote \` instead of forward quote. ksqlDB differentiate between back, forward and double quote. In my case, User-agent has to be enclosed with \` quote. In a delimited format with space, you can technically used any column name, but in JSON, you if you like to select the key in your json that also uses "-", then the back quote is useful.

1. create stream botnet (Date VARCHAR, Time VARCHAR, Method VARCHAR, URI VARCHAR,ClientIP VARCHAR, `User-agent` VARCHAR) WITH (kafka_topic='webserver', key_format='delimited', value_format='delimited', VALUE_DELIMITER='SPACE');

2. Once the stream is created, you can query the data using SQL select. ksql does not support ORDER BY
3. And remember to see data from the earliest, you should either set this on ksql-cli SET 'auto.offset.reset' = 'earliest'; or in the ksql-server.properties

select * from botnet; </br>
Display all the logs from the topic

select count(*) from botnet where `User-agent` LIKE '%python%' emit changes; </br>
Display and count based on User-agent column which contains the string "python"

select * from botnet where `User-agent` NOT LIKE '%Mozilla%'; </br>
Display all data from topic where the Column User-agent does not contain string Mozilla

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
With delimiter file with SPACE, you will have to define all the columns. With JSON, you can select which keys you like to be part of the stream to be included, offering greater flexibility of not importing all data. With Delimiter format log, if you have space as the "delimit" format, care should be taken as any space in your data log such as description can be mistook by ksql. In my case, User-agent field for HTTP contains long description with space, kSQL mistook it as another column. I end up cleaning up my logs with sed and awk to have meaningful analysis. ksqlDB only works on structure data.

## Creating Materialised View or Table ##
ksqlDB supports creating a Materialised View or table in rocksdb that aids querying data from the table much faster. We can create a table with interested datasets by using the select statement.

ksql> create table botnettable as select count(*) as count, `User-agent`,ClientIP,URI from botnet GROUP BY `User-agent`,ClientIP,URI;
                                
```
-------------------------------------------
 Created query with ID CTAS_BOTNETTABLE_45 
 ----------
 ```

After the table is created, you can query from the table where count is more than 1. This cannot be done on streams. However, you can query streams that match on specified values like matching strings.

ksql> select * from botnettable where count >1;
```
+--------------------------+--------------------------+--------------------------+--------------------------+
|User-agent                |CLIENTIP                  |URI                       |COUNT                     |
+--------------------------+--------------------------+--------------------------+--------------------------+
|                          |103.115.120.250           |/                         |2                         |
|                          |104.84.150.77             |/vp-assets/css/skel.css   |4                         |
|                          |104.84.150.77             |/vp-assets/css/style-wide.|3                         |
|                          |                          |css                       |                          |
|                          |104.84.150.77             |/vp-assets/js/init.js     |4                         |
|                          |104.84.150.77             |/vp-assets/js/jquery.count|5                         |
|                          |                          |down360.js                |                          |
|                          |104.84.150.77             |/vp-assets/js/jquery.dropo|2                         |
|                          |                          |tron.min.js               |                          |
|                          |104.84.150.77             |/vp-assets/js/jquery.min.j|5                         |
|                          |                          |s                         |                          |
|                          |104.84.150.77             |/vp-assets/js/skel-layers.|4                         |
|                          |                          |min.js                    |                          |
|                          |104.84.150.77             |/vp-assets/js/skel.min.js |2                         |
|                          |104.84.150.77             |/vp-assets/waiting.png    |3                         |
|Firefox                   |194.110.203.47            |/database/dbdump.rar      |2                         |
|Firefox                   |194.110.203.47            |/include/config.php.bck   |2                         |
|Go-http-client/1.1        |111.7.96.134              |/                         |10                        |
|Go-http-client/1.1        |111.7.96.162              |/                         |12                        |
|GuzzleHttp/7              |185.70.41.32              |/.well-known/openpgpkey/po|3                         |
|                          |                          |licy                      |                          |
|HaxerMen                  |107.175.69.18             |/web_shell_cmd.gch        |3                         |
|Hello                     |141.98.10.56              |/                         |54                        |
|Hello                     |141.98.11.39              |/                         |3                         |
|IDBTE4M                   |13.76.177.48              |/                         |2                         |
|IDBTE4M                   |13.76.177.48              |/.env                     |2                         |
|IonCrawl                  |212.227.216.16            |/                         |2                         |
|Java/1.8.0_144            |60.214.64.3               |/dns-query                |16                        |
|Java/1.8.0_275            |108.61.186.155            |/owa/auth/logon.aspx      |2                         |
|Java/1.8.0_282            |162.218.65.10             |/                         |2                         |
|Java/1.8.0_321            |162.255.87.136            |/                         |2                         |
|python-requests/2.28.1    |86.98.144.188             |/?query=show%20status     |2                         |
|python-requests/2.28.1    |86.98.144.188             |/catalog-portal/ui/oauth/v|2                         |
|                          |                          |erify?error=&deviceUdid=%2|                          |
|                          |                          |4%7B%22freemarker.template|                          |
|                          |                          |.utility.Execute%22%3Fnew%|                          |
|                          |                          |28%29%28%22cat%20%2Fetc%2F|                          |
|                          |                          |passwd%22%29%7D           |                          |
|python-requests/2.28.1    |86.98.144.188             |/conf_mail.php            |2                         |
|python-requests/2.28.1    |86.98.144.188             |/fileupload/toolsAny      |4                         |
|python-requests/2.28.1    |86.98.144.188             |/search/index.php?keyword=|2                         |
|                          |                          |1%2527%2520%2561%256e%2564|                          |
|                          |                          |%2520%2528%2565%2578%2574%|                          |
|                          |                          |2572%2561%2563%2574%2576%2|                          |
|                          |                          |561%256c%2575%2565%2528%25|                          |
|                          |                          |31%252c%2563%256f%256e%256|                          |
|                          |                          |3%2561%2574%2528%2530%2578|                          |
|                          |                          |%2537%2565%252c%2528%2573%|                          |
|                          |                          |2565%256c%2565%2563%2574%2|                          |
|                          |                          |520%2575%2573%2565%2572%25|                          |
|                          |                          |28%2529%2529%252c%2530%257|                          |
|                          |                          |8%2537%2565%2529%2529%2529|                          |
|                          |                          |%2523                     |                          |
|python-requests/2.28.1    |86.98.144.188             |/vpns/portal/bc46bd04.xml |2                         |
|python-requests/2.28.1    |86.98.144.188             |/yyoa/common/js/menu/test.|2                         |
|                          |                          |jsp?doType=101&S1=(SELECT%|                          |
|                          |                          |20MD5(1))                 |                          |
|python-requests/2.28.1    |86.98.144.188             |/zabbix                   |2                         |
|python-requests/2.28.1    |86.98.144.188             |/ztp/cgi-bin/handler      |4                         |
```

ksql> show tables;
```

 Table Name  | Kafka Topic | Key Format | Value Format | Windowed 
------------------------------------------------------------------
 BOTNETTABLE | BOTNETTABLE | DELIMITED  | DELIMITED    | false    
------------------------------------------------------------------
ksql> 
```

## Stream - Table, Stream - Stream, Table - Table JOIN ##
ksqlDB supports JOIN between 2 streams, Stream - table (the reverse does not work), and 2 table joins.
Something to note table-stream joins are not supported; only stream-table joins, Stream-table will require a key in table, join to the key.

Stream-stream join will require WITHIN to specified the time
1. Executing Stream - Stream JOIN and comparing, note the LEFT OUTER JOIN key word to join with another stream, using condition IP of stream example and ClientIP in botnet stream.</br>

ksql> select * from example a LEFT OUTER JOIN botnet b WITHIN (0 SECONDS, 7 DAYS)  ON a.IP = b.ClIENTIP emit changes;
```
+----------+----------+----------+----------+----------+----------+----------+----------+----------+
|A_DATE    |A_TIME    |A_IP      |B_DATE    |B_TIME    |B_METHOD  |B_URI     |B_CLIENTIP|B_User-age|
|          |          |          |          |          |          |          |          |nt        |
+----------+----------+----------+----------+----------+----------+----------+----------+----------+
|2023/05/23|18:22:52  |64.62.197.|2021-04-03|09:21:00  |GET       |/         |64.62.197.|          |
|          |          |2         |          |          |          |          |2         |          |
|2023/05/23|18:22:52  |64.62.197.|2021-04-07|08:44:36  |GET       |/         |64.62.197.|          |
|          |          |2         |          |          |          |          |2         |          |
|2023/04/28|05:29:14  |185.180.14|2021-09-18|02:48:56  |GET       |/remote/lo|185.180.14|Mozilla/5.|
|          |          |3.136     |          |          |          |gin       |3.136     |0 10.0;   |
|2023/05/13|15:46:11  |64.62.197.|2022-10-27|13:25:13  |GET       |/favicon.i|64.62.197.|Mozilla/5.|
|          |          |200       |          |          |          |co        |200       |0 Linux   |
|2023/04/30|11:54:59  |74.82.47.2|2022-11-12|18:52:27  |GET       |/         |74.82.47.2|Mozilla/5.|
|          |          |4         |          |          |          |          |4         |0 x86_64; |
|2023/05/20|20:05:58  |185.180.14|2022-11-16|06:52:30  |GET       |/         |185.180.14|Mozilla/5.|
|          |          |3.18      |          |          |          |          |3.18      |0 10.0;   |
|2023/05/20|20:05:58  |185.180.14|2022-11-16|06:52:44  |GET       |/webfig/  |185.180.14|Mozilla/5.|
|          |          |3.18      |          |          |          |          |3.18      |0 10.0;   |
|2023/05/20|20:05:58  |185.180.14|2022-11-16|06:52:57  |GET       |/solr/    |185.180.14|Mozilla/5.|
|          |          |3.18      |          |          |          |          |3.18      |0 10.0;   |
|2023/05/21|03:50:52  |185.180.14|2022-12-07|18:46:23  |GET       |/webfig/  |185.180.14|Mozilla/5.|
|          |          |3.11      |          |          |          |          |3.11      |0 10.0;   |
|2023/05/09|10:00:01  |64.62.197.|2022-12-25|22:42:10  |GET       |/api/v2/st|64.62.197.|Mozilla/5.|
|          |          |91        |          |          |          |atic/not.f|91        |0 10.0;   |
|          |          |          |          |          |          |ound      |          |          |
```
##### Some kafka tricks #####
Deleting messages from kafka topics
kafka-delete-records.sh --bootstrap-server 192.168.0.100:9092  --offset-json-file delete-records.json

Dumping records from the start
kafka-console-consumer.sh --bootstrap-server 192.168.0.100:9092 --topic webserver --from-beginning

Ingesting records
kafka-console-producer.sh --bootstrap-server 192.168.0.100:9092 --topic webserver </yyy/xxx.log

### To be Work On
Will include create stream using json format with struct and array. Lost all the configs when I have to re-install.
