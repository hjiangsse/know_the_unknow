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

### concurrency problem:
On a 8 core mechine, sequence read is better than cocurrent read. (How about a 120 core mechine? YTM!)  
Update: What you see is wrong! please use more big block size!

### concurrent test:
#---------------------------------------------  
poolsize = core_number / 2  
blocksize = core_numer * 2  

time:  31.494286060333252  
time:  30.764371395111084  
time:  31.367915153503418  
time:  31.27149200439453  
time:  30.45405125617981  
time:  30.330586433410645  
time:  30.621195793151855  
time:  31.257272243499756  
time:  30.622308015823364  
time:  31.576156616210938  
avg_tm:  30.975963497161864  

#---------------------------------------------  
poolsize = core_number / 4  
blocksize = core_numer * 2  

time:  36.21045517921448  
time:  34.0615930557251  
time:  29.471998691558838  
time:  28.961904287338257  
time:  42.42263865470886  
time:  30.65553307533264  
time:  36.3575325012207  
time:  29.724627256393433  
time:  30.105841398239136  
time:  37.22924876213074  
avg_tm:  33.52013728618622  

#---------------------------------------------  
poolsize = core_number / 1  
blocksize = core_numer * 2  

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

### Aggregate Functions
### Aggregating Merge Tree
