# Build the AMI
https://github.com/TachyonNexus/golluxio/blob/master/img-creator/README.md
In `golluxio/img-creator`, run the following command to build the new AMI and launch test instance
```
bin/img-creator packer build --launchTestInstance
```

After it finished, log in to the new instance

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

and then restart all these services to pick up changes
sudo systemctl restart hadoop
sudo systemctl restart hcat
sudo systemctl restart presto


