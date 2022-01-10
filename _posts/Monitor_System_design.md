
# Overview
## What kind of problem Incident Platform Solve
For example, there is an online payment company which provides payment services for merchants and customers. The company categorizes all businesses into offline business, online business. 
And offline business can be categorized into:
 - restaurants
 - supermarkets  
 - schools
 - hospitals, etc. 

Online business can be categoried into: 
 - E-commerce business
 - E-Game
 - Tiket and booking
 - Traveling, etc.

In each category there are many merchants, each of merchants built an app to provide services, and customers pays with the company's app to buy the service or products.
But sometimes, the merchants' apps will fail to load, or the customers can't login to the app, or the payment does't work. Each these failure will cause the lose of
money and the patience of customers. In order to solve these problems, the company need to build a incident platform.

## The Platform Goal Definition
The business goal of Incident Platform: Improve the Stability of the Whole System（稳定性）.
There are various ways to calculate Stability:
 - UV Stability = 1 - ImpactUsers/WholeUsers, for online business (影响用户数)
 - PV Stability = 1 - ImpactPv/WholePv, for web services, some pages work, some not （影响服务次数）
 - Time Stability = 1 - NotWorkingTime/WholeTime, for hardware （影响时长）

In practice, the first one is the mostly chosed.

In order to achieve the goal, there are 2 sub KPIs:
 - Reduce the number of incidents (故障数). The ways:
   - Pre-Online Test: Performance Test
   - Periodic Risk Inspection: Daily or weekly inspect the system based on risk rules
   - For Security: Red Army vs Blue Army, White Hat Bonus, etc
 
 - Reduce the average time cost of incidents（平均故障时长）
   - Build emergency plan to Standardize incident handling processes
   - Tools to help automatically handling process: Auto Alert, Auto call people, Auto create groups, etc.

In practice, the second way to more reliable and predictable than the first one.

## International Standard
ITIL V3.0 (Information Technology Infrastructure Library) By IBM, HP, Government, etc.
The part of Incident Management in ITIL.


# Use Case
## The Key Concept Definition
The system input data can be categorized into:
 - ***RealTime Data（实时数据）***. Only logs, only handled by Flink.
 - ***Batch Data（批处理数据）***. SQL table data, Mysql, Hive, Kylin; ElasticSearch; Hbase.

So, if a new app needs to served by our platform, it only needs to integrate with our data interface:
 - Write the code to add logs and config the format infomation. If a standard log format is used, the config process can be omitted.
 - Config the name and schema of related table data.

Then all is done.

For example, in the above company, if some of the merchant app fail, those merchant's payment service will be influenced. So the company wants to monitor this kind of situation, the following steps taken.

### Step 1: App business side to choose metrics and monitor rules
- The first step for monitor is to ***choose the metrics(选择指标)***, including ***Realtime Metrics（实时指标）*** based on realtime data(实时数据), ***NON-Realtime Metrics(非实时指标)*** based on batch data(批处理数据). 
- The Second step is to build ***monitor rules（监控规则）*** based on metrics, normally each metric should have one monitor rule.

For example, we create 2 metrics:
 - ***merchant payment count by minute(商户交易笔数每分钟)***：Realtime metric
 - ***merchant payment count by day(商户每天交易笔数)***：NON-realtime metric

And then create 3 monitor rules:
- Merchant payment count less than 100 in last 3 miniutes: realtime fixed monitor rule
- Merchant payment count suddenly drops: realtime smart monitor rule
- On 8:00am, check merchant payment yesterday count less than 1000: non-realtime fixed monitor rule

```
 A metric is made of 3 components: 
 - ***Dimension（维度）***: monitor who. If no dimension is defined, we choose ***sys(系统维度)*** as the default.
 - ***Time(时间）***: monitor when.
 - ***Value(指标值）***: monitor value.

For example:
- ***Merchant payment count by minute*** metric has merchantID as dimension, every minute as time, 300 or 400 as value.
- ***Whole payment count by minute*** metric has sys as dimension, every minute as time, 3000 or 4000 as value.
```
```
 A monitor rule is made of several components:
 - Valid time（运行时间）. For example, only run at week days except weekends.
 - Detect period(运行周期)：every minute，daily，weekly, etc.
 - Dimension set: For example, manually input: only monitor merchants with ID 222,333; or sys as default; or imported from a table (select merchant_id from xx)
 - Dimension blacklist set.
 - Rule type(规则类型). Fixed rule(固定规则) or smart rule（智能规则）.
 - Expression set: one or more expressions combined together with AND/OR logic. Fixed type expression has several property, for example, ***merchant payment count less than 100 in last 3 miniutes***:
   - Metric (指标): merchant payment count in minute;
   - Metric time span（指标时间跨度）: last 3 min;
   - Metric time span operator(指标时间跨度运算符): sum, can also be average, etc;
   - Metric compare operator(指标比较运算符): less than, can also be large, equal, etc;
   - Threshold(阈值)：100，can also be percent 30%, etc;

 - Smart rule has several properties: for example, ***merchant payment count suddenly drops***:
   - Metric:
   - Smart operator(智能运算符): suddenly drop（快速下跌）, can also be slowly drop（缓慢下跌）, suddenly rise（快速上升）, slowly rise（缓慢上升）, etc
 - Non-disturb rule(防打扰规则). For example, alert ***3 times*** max in ***24 hours***.
 ```
 
 ### Step 2: App developers to Add monitor logs and provide batch data
 #### Add monitor log
 The developers decide to add this simple log after each payment process:
 ```
 time,paymentId,merchantId,customerId,amount,success or not
 ```
 For advanced usage, adding monitor log(埋点) can be based on ***Common Log Formatter(统一埋点规范)***. The log formatter can be categorized into frontend and bakened. The frontend formatter can refer to [Ali SPM standard](https://blog.blublu.site/2021/04/spm-scm-model/index.html).
 ![image](https://user-images.githubusercontent.com/4775215/148730426-0967b0a9-cbc1-4136-a35b-81bd41abcb9d.png)

#### Provide batch data
The developers need to provide a mysql table which contains the merchant information, for example, merchantId, merchantName, city, address, business_category, etc. This information will make alert messages more readable.

### Step 3: Notification config
We need to config:
- Who will receive the notification: commonly this data will be provided by HR system or CMDB;
- Message channel: SMS, IM, phone, etc;
- Message template: for each channel, the template will be slightly different, for example, phone call template should remove merchantId 2334233.
- When: 8:00am - 8:00pm
- Non-disturb rule(防打扰规则). For example, alert ***3 times*** max in ***24 hours***.
- Data Dashboard link: to check what is happened
- Incident dashboard link: to handle incident

### Finally: Incident Data analysis Dashboard
This is a report dashboard system, we can build all kinds of charts, tables, tabs just by draging and selecting the metrics. When incidents happend, people will visit this dashboard to find what happens.

# System Architecture

## The core design rules
The system input data can be categorized into:
 - ***RealTime Data（实时数据）***. Logs, often handled by Flink.
 - ***Batch Data（批处理数据）***. SQL table data, Mysql, Hive, Kylin; ElasticSearch; Hbase.

So, if a new app needs to served by our platform, it only needs to integrate with our data interface:
 - Write the code to add logs and config the format infomation. If a standard log format is used, the config process can be omitted.
 - Config the name and schema of related table data.

Then all is done.

## Metric Management System
Goal: 
- Metric data config management: Create/Update/Delete
- Metric data storage: Hbase/OPENTSDB/Kylin/Elasticsearch
- Metric data service: provide apis for other systems to query data

Preset: App developers have add monitor logs and put logs into log fetch system.

Upstream system: log fetch system/MQ, or Mysql,Elasticsearch.
Downstream: all other systems.


### Use case
The user needs to select which app he belongs to before login to metric data system.

#### Case 1: Data source Create
##### Case 1.1: Realtime data source create
User should fill in these configs:
- Data source name
- Data source type: log system, MQ
- Data table path: for log system, this could be the app name and log file path; for MQ, this could be channel name;
- Data parse script: split by ```,```, json formatter, Groovy parse script, etc.
- Data schema: for each parsed field, need to give a name, and data type (string, int, float)
 
##### Case 1.2: Batch data source create
User should fill in these configs:
- Data source name
- Data source type: Mysql, Elasticsearch, etc.
- Data table path: database name and table name;
- Data fetch sql: optional, use the sql to filter unused fields or records
- Data schema: for each parsed field, need to give a name, and data type (string, int, float)

#### Case 2: Metric create
User should fill in these configs:
- Metric name
- Data source id above
- Dimension field: one or more fields in schema may be joined by a special delimiter (@#) to form a string key; if no field is selected, the default sys is used.
- Value field: numberic field to get sum; for string id field to calculate UV/PV; or count as default;
- PV/UV: for for string id field to calculate UV/PV;
- Value time span: one minute;
- Value operator: sum/average;

#### Case 2: Metric data preview
After the config is done, users can check whether the ouput data is expected.
### Demo Design
TODO

### Implementation
#### Realtime metric create
After the metric config is done, a data fetch job is created:
- for realtime metric, a Flink job is created; 
- for batch data metric, a Datawarehouse job is created.

The job will parse data and store data:
- For metric data: store in Hbase, with the fields: （metricId, dimension）as rowkey, timestamp, value, value_max, value_min, value_mid. The last 3 values are used by smart algorithm to build models.
- For full data storge: every record is stored in Kylin, used by Impact Analysis()

### QA

