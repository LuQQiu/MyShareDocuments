# Build the AMI
Refer to the following link to set up img creator.
https://github.com/TachyonNexus/golluxio/blob/master/img-creator/README.md

In `golluxio/img-creator`, run the following command to build the new AMI and launch test instance
```
bin/img-creator packer build --launchTestInstance
```
It will first build a `packer-build` instance to build the AMI, and then use the new AMI to launch a actual test instance.

After it finished, log in to the new test instance

# Support Presto on S3 workflow
By default, the instance support presto on Alluxio workflow.
But it did not support presto on S3 workflow.

## Hadoop `core-site.xml`
```
    <property>
        <name>fs.alluxio.impl</name>
        <value>alluxio.hadoop.FileSystem</value>
    </property>
    <property>
        <name>fs.s3a.access.key</name>
        <value>****</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>****</value>
    </property>
```

## Hive `hive-site.xml`
```
    <property>
        <name>fs.s3a.access.key</name>
        <value>****</value>
    </property>
    <property>
        <name>fs.s3a.secret.key</name>
        <value>****</value>
    </property>
```

Need to download the `hadoop-aws-2.8.5.jar` and `aws-java-sdk-bundle-1.11.271.jar` and put in the lib directory.

## Presto `/etc/catalog/hive.properties`
```
hive.s3.use-instance-credentials=false
hive.s3.aws-access-key=***
hive.s3.aws-secret-key=****
```

## restart all these services to pick up changes
```
sudo systemctl restart hadoop
sudo systemctl restart hcat
sudo systemctl restart presto
```

# Create hive tables with S3 data
```
hive -f createS3DirectTables.sql
```

# Create hive tables with Alluxio data
```
alluxio fs mount --readonly --option aws.accessKeyId=**** --option aws.secretKey=*** /s3 s3a://autobots-tpcds-test-data/parquet/scale100
hive -f createAlluxioS3Tables.sql
```

# Run presto queries
Use `presto --server localhost:8080 --catalog hive --debug` to trigger presto CLI.
Use `use alluxio;` or `use s3;` to choose different schema.
and then run your favorite queries!


