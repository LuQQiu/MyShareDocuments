
1. change alluxio configuration
`alluxio.worker.tieredstore.level0.dirs.quota=40GB`
`alluxio.user.block.size.bytes.default=128MB`

2. change hive configuraion
cp /opt/hive/conf/hive-env.sh.template hive-env.sh
/opt/hive/conf/hive-env.sh
comment out the following lines
```
 if [ "$SERVICE" = "cli" ]; then
   if [ -z "$DEBUG" ]; then
     export HADOOP_OPTS="$HADOOP_OPTS -XX:NewRatio=12 -Xms10m -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:+UseParNewGC -XX:-UseGCOverheadLimit"
   else
     export HADOOP_OPTS="$HADOOP_OPTS -XX:NewRatio=12 -Xms10m -XX:MaxHeapFreeRatio=40 -XX:MinHeapFreeRatio=15 -XX:-UseGCOverheadLimit"
   fi
fi
export HADOOP_HEAPSIZE=1024
```
and add
```
export HIVE_OPTS="-hiveconf mapreduce.map.memory.mb=4096 -hiveconf mapreduce.reduce.memory.mb=5120"
```

3. change presto configuration
change `/opt/presto/etc/config.properties`
```
query.max-memory=20GB
query.max-memory-per-node=20GB
query.max-total-memory-per-node=20GB
```
change `/opt/presto/etc/jvm.config`
```
-Xmx35G
```

4. restart all the things and mount the S3 data set
```bash
alluxio fs mount --readonly --option aws.accessKeyId=**** --option aws.secretKey=*** /s3 s3a://autobots-tpcds-test-data/parquet/scale100
```

5. put a small table `promotion` in the Alluxio with path
`alluxio://localhost:19998/promotion`
the table size is 54KB.

```bash
wget https://autobots-tpcds-test-data.s3.amazonaws.com/parquet/scale100/promotion/part-00000-799bb353-4be8-4189-b02e-8ccb71463cbf-c000.snappy.parquet

alluxio fs mkdir /promotion

alluxio fs copyFromLocal part-00000-799bb353-4be8-4189-b02e-8ccb71463cbf-c000.snappy.parquet /promotion/
```

6. create the external Alluxio table in hive
create the store_sales, store_returns, web_sales, web_returns with data in Alluxio on S3 directory
using `hive -f createTestTables.sql`

create the promotion with data in Alluxio promotion data
using `hive -f createPromotionTable.sql`

7. Run presto query for twice and see the time difference

way 1 (suggested): trigger presto sql `presto --server localhost:8080 --catalog hive`
and copy the queries in  `prestoQuery.sql`

This way presto will show the detailed progress and time for each query


way 2: directly run `presto --server localhost:8080 --catalog hive -f prestoQuery.sql`
User may feel the query stuck as it will take several minutes 

