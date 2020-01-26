# apk8s-hive

This repository is for the demonstration of Apache Hive utilizing a MySQL database for metadata storage, specifically for the projection of schema atop S3 object storage.

Custom Hive container build instructions.

```shell script
# Download Apache Hive
curl -L http://mirror.cc.columbia.edu/pub/software/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz -o ./src/apache-hive-3.1.2-bin.tar.gz

# Download Apache Hadoop
curl -L http://archive.apache.org/dist/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz -o ./src/hadoop-3.1.2.tar.gz

# Uncompress both
tar -xzvf ./src/apache-hive-3.1.2-bin.tar.gz -C ./src
tar -xzvf ./src/hadoop-3.1.2.tar.gz -C ./src

# Add Jars
./deps.sh

# create container
docker build -t apk8s-hive-s3m:3.1.2 .

# local test (remote S3)
docker-compose up

# connect to hive CLI
docker exec -it apk8s-hive_hive-metastore_1 /opt/hive/bin/hive

# create table
create table donors (
    email string,
    name string,
    type string,
    birthday string,
    state string)
row format delimited fields terminated by ','
lines terminated by "\n" location 's3a://upload/'
tblproperties ("skip.header.line.count"="1");
```

Add container to registry:
```shell script
docker tag apk8s-hive-s3m:3.1.2 apk8s/hive-s3m:3.1.2-1.0.0
docker push apk8s/hive-s3m:3.1.2-1.0.0
```
