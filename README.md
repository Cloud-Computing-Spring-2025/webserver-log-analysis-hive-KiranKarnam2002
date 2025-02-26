# Web-Server-Log-Analysis
# Project Overview
This project focuses on analyzing web server logs using Apache Hive in a Hadoop environment. The analysis includes counting total requests, analyzing HTTP status codes, identifying most visited pages, tracking traffic sources, detecting suspicious activity, and analyzing traffic trends. Partitioning is implemented for performance optimization.
# Implementation Approach
The approach follows a structured methodology:
## Setting up Hadoop and Hive: Ensuring all services are running.
*Creating a Hive database and tables:* Defining an external table and a partitioned table.

*Loading data:* Copying CSV logs into HDFS and registering them in Hive.

*Executing queries:* Running analysis tasks with HiveQL.

*Optimizing with partitioning:* Enhancing performance by partitioning data by HTTP status.

*Exporting results:* Saving query outputs for reporting.
## Execution Steps
### 1. *Start the Hadoop Cluster*
Run the following command to start the Hadoop cluster:

bash
docker compose up -d

### 2. *Step 2: Create Database and Tables*
Create a new database:
bash
CREATE DATABASE web_logs;
USE web_logs;

Define an external table:
bash
CREATE EXTERNAL TABLE web_logs (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/web_logs/';

### 3. *Run Queries for Analysis*

*1. Count Total Web Requests*
bash
SELECT COUNT(*) AS total_requests FROM web_logs;

*2. Analyze HTTP Status Codes*
bash
SELECT status, COUNT(*) AS count FROM web_logs GROUP BY status ORDER BY count DESC;

*3. Identify Most Visited Pages*
bash
SELECT url, COUNT(*) AS visits FROM web_logs GROUP BY url ORDER BY visits DESC LIMIT 3;

*4. Traffic Source Analysis*
bash
SELECT user_agent, COUNT(*) AS count FROM web_logs GROUP BY user_agent ORDER BY count DESC;

*5. Detect Suspicious Activity*
bash
SELECT ip, COUNT(*) AS failed_requests
FROM web_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING failed_requests > 3;

*6. Analyze Traffic Trends*
bash
SELECT SUBSTR(`timestamp`, 0, 16) AS minute, COUNT(*) AS request_count
FROM web_logs
GROUP BY SUBSTR(`timestamp`, 0, 16)
ORDER BY minute;

# Step 4: Implement Partitioning for Optimization #
*Create Partitioned Table* 
bash
CREATE TABLE web_logs_partitioned (
    ip STRING,
    `timestamp` STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

bash
SET hive.exec.dynamic.partition.mode=non-strict;

bash
INSERT OVERWRITE TABLE web_logs_partitioned PARTITION (status)
SELECT ip, "timestamp", url, user_agent, status FROM web_logs;

# Step 5: Export Query Results #
*Save total request count to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/total_requests'
SELECT COUNT(*) FROM web_logs;


*Save Status Code Analysis to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Status_Code_Analysis'
SELECT status, COUNT(*) AS count FROM web_logs GROUP BY status ORDER BY count DESC;

*Save Most Visited Pages to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Most_Visited_Pages'
SELECT url, COUNT(*) AS visits FROM web_logs GROUP BY url ORDER BY visits DESC LIMIT 3;


*Save Traffic Source Analysis to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Traffic_Source_Analysis'
SELECT user_agent, COUNT(*) AS count FROM web_logs GROUP BY user_agent ORDER BY count DESC;


*Save Suspicious IP Addresses to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Suspicious_IP_Addresses'
SELECT ip, COUNT(*) AS failed_requests
FROM web_logs
WHERE status IN (404, 500)
GROUP BY ip
HAVING failed_requests > 3;

*Save Traffic Trend Over Time to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Traffic_Trend_Over_Time'
SELECT SUBSTR(`timestamp`, 0, 16) AS minute, COUNT(*) AS request_count
FROM web_logs
GROUP BY SUBSTR(`timestamp`, 0, 16)
ORDER BY minute;


*Save Partitioning to HDFS:*
bash
INSERT OVERWRITE DIRECTORY '/user/hive/output/Partitioning'
SELECT ip, "timestamp", url, user_agent, status FROM web_logs;


### 6. *Setup HDFS*

Access the hive server:

bash
docker exec -it hive-server /bin/bash


Navigate to the output directory:

bash
hdfs dfs -get /user/hive/output /tmp/output


Hive container to check the files
bash
ls -l /tmp/output


exit now from the container
bash
exit

current working directory
bash
pwd

bash
docker cp hive-server:/tmp/output /workspaces/webserver-log-analysis-hive-saivenkatchoppara


## Challenges Faced ##

*Handling Incorrect Data Formats:* Ensured CSV data was properly structured before loading.

*Query Performance:* Used partitioning to optimize queries involving status code filtering.

*Hive Table Syncing:* Used MSCK REPAIR TABLE to refresh partitions dynamically.

## Sample Input and Expected Output ##
*Sample Log Entries*
bash
ip,timestamp,url,status,user_agent
192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1
192.168.1.10,2024-02-01 10:18:00,/home,500,Mozilla/5.0
192.168.1.15,2024-02-01 10:19:00,/products,404,Chrome/90.0


## Expected Query Results ##
*Total Requests*
bash
Total Requests: 100

*Status Code Analysis*
bash
200: 80
404: 10
500: 10

*Most Visited Pages*
bash
/home: 50
/products: 30
/checkout: 20

*Traffic Source Analysis*
bash
Mozilla/5.0: 60
Chrome/90.0: 30
Safari/13.1: 10

*Suspicious IP Addresses*
bash
192.168.1.10: 5 failed requests
192.168.1.15: 4 failed requests

*Traffic Trends*
bash
2024-02-01 10:15: 5 requests
2024-02-01 10:16: 7 requests

## Conclusion ##
 
This project successfully analyzes web server logs using Apache Hive and optimizes query performance with partitioning. The structured approach ensures efficiency in data processing, making it scalable for large datasets.
