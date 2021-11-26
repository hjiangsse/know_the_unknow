# clickhouse
## Some basic ops
### install and start(debian)
sudo apt-get install apt-transport-https ca-certificates dirmngr  
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4  
echo "deb https://repo.clickhouse.com/deb/stable/ main/" | sudo tee /etc/apt/sources.list.d/clickhouse.list  
sudo apt-get update  
sudo apt-get install -y clickhouse-server clickhouse-client  
clickhouse-client  

sudo service clickhouse-server start  
or  
sudo /etc/init.d/clickhouse-server start  

### allow remote mechine connect to this server
vim /etc/clickhouse-server/config.xml  
/search listen-host  
then uncomment the line  
### create a table
``` python
    create_query = '''
                    CREATE TABLE tutorial.frame_new_how
                    (
                        `EventTime` FixedString(19),
                        `Close` Float64,
                        `Open` Float64,
                        `Avg` Float64, 
                        `Field1` Float64,
                        `Field2` Float64,
                        `Field3` Float64,
                        `Field4` Float64,
                        `Field5` Float64
                    )
                    ENGINE = MergeTree()
                    PRIMARY KEY EventTime
                    ORDER BY (EventTime)
    '''
    create_res = client.execute(create_query)
    print(create_res)
```
### insert csv file into a table
clickhouse-client --password='jiang186212' --format_csv_delimiter="," --max_insert_block_size=100000  --query="INSERT INTO \ tutorial.frame_new_how FORMAT CSV" < frame.csv

### python package(clickhouse-driver)
pip install clickhouse-driver -i https://pypi.tuna.tsinghua.edu.cn/simple

### benchmark:
clickhouse-benchmark --query="select * from factor_database.factor_big_table1" -i 10 -h 192.168.222.220

### clickhouse-cpp build and install
git clone the code base  
mkdir build  
cd build  
cmake ..  
make  
sudo make install  

## The real-time clickhouse
Since the updates in clickhouse are asynchronous, we can't see the updates of records in realtime.
So we have to insert a new modified record instead, and we must find a way to find the lastest one.

### Replacing Merge Tree
1. create a table use "ReplacingMergeTree" as the engine
```
CREATE TABLE alerts(
  tenant_id     UInt32,
  alert_id      String,
  timestamp     DateTime Codec(Delta, LZ4),
  alert_data    String,
  acked         UInt8 DEFAULT 0,
  ack_time      DateTime DEFAULT toDateTime(0),
  ack_user      LowCardinality(String) DEFAULT ''
)
ENGINE = ReplacingMergeTree(ack_time)
PARTITION BY tuple()
ORDER BY (tenant_id, timestamp, alert_id);
```
In the previous table, two "rows" with the same (tenant_id, timestamp, alert_id) will be regard as 
the same record, the "newer" version of the record will replace the old one. The 'newness' is 
determined by 'ack_time' in this case;

![table created](./pics/database/replacing_merge_tree.png)

fill the table with some data:
```
INSERT INTO alerts (tenant_id, alert_id, timestamp, alert_data) SELECT
    toUInt32((rand(1) % 1000) + 1) AS tenant_id,
    randomPrintableASCII(64) AS alert_id,
    toDateTime('2020-01-01 00:00:00') + (rand(2) % ((3600 * 24) * 30)) AS timestamp,
    randomPrintableASCII(1024) AS alert_data
FROM numbers(10000000)
```

Then change 90% rows, not update but insert new rows:
```
INSERT INTO alerts (tenant_id, alert_id, timestamp, alert_data, acked, ack_user, ack_time)
SELECT tenant_id, alert_id, timestamp, alert_data, 
  1 as acked,
  concat('user', toString(rand()%1000)) as ack_user,
  now() as ack_time
FROM alerts WHERE cityHash64(alert_id) % 99 != 0;
```
The insert result:
![table insert](./pics/database/insert_new_rows.png)
![table insert](./pics/database/count_after_insert.png)

Now we have more than 1800w rows(some are old, some are new). If we want to see the newest rows, use the *FINAL* select
```
SELECT count() FROM alerts FINAL
```
![table select](./pics/database/final_select_count.png)

select use one of primary key and filter
```
SELECT count(), sum(cityHash64(*)) AS data FROM alerts FINAL WHERE (tenant_id = 451) AND (NOT acked)
```
![table select](./pics/database/select_with_primary_key.png)

we can see, select with a primary key and a filter can much decrease the "searched rows".
why is it fast this time? The difference is in the filter condition. 
‘tenant_id’ is a part of a primary key, so ClickHouse can filter data before FINAL. 
In this case, *ReplacingMergeTree* becomes efficient.

### Aggregate Functions
Let's check the previous select again! 
```
SELECT count(), sum(cityHash64(*)) AS data FROM alerts FINAL WHERE (tenant_id = 451) AND (NOT acked)
```
Let see how we can use aggregate functions(聚集函数) to accelerate the select process.
check the "update insert query" again.
```
INSERT INTO alerts (tenant_id, alert_id, timestamp, alert_data, acked, ack_user, ack_time)
SELECT tenant_id, alert_id, timestamp, alert_data, 
  1 as acked,
  concat('user', toString(rand()%1000)) as ack_user,
  now() as ack_time
FROM alerts WHERE cityHash64(alert_id) % 99 != 0;
```
*acked* change from 0 to 1
*ack_user* change from '' t0 'userxxx'
*ack_time* change from 0 to Now
all change are "inscreasing", we can use *max* aggregate functions

```
SELECT count(), sum(cityHash64(*)) data FROM 
  SELECT tenant_id, alert_id, timestamp, 
    any(alert_data) alert_data, 
    max(acked) acked, 
    max(ack_time) ack_time,
    max(ack_user) ack_user
  FROM alerts
  GROUP BY tenant_id, alert_id, timestamp
WHERE tenant_id=451 AND NOT acked;
```
### Aggregating Merge Tree
```
DROP TABLE alerts_amt_max;
CREATE TABLE alerts_amt_max (
  tenant_id     UInt32,
  alert_id      String,
  timestamp     DateTime Codec(Delta, LZ4),
  alert_data    SimpleAggregateFunction(max, String),
  acked         SimpleAggregateFunction(max, UInt8),
  ack_time      SimpleAggregateFunction(max, DateTime),
  ack_user      SimpleAggregateFunction(max, LowCardinality(String))
)
Engine = AggregatingMergeTree()
ORDER BY (tenant_id, timestamp, alert_id);
```
The aggrating merge tree will keep data size smaller.
