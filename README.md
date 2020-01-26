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
Kubernetes Config:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hive
  namespace: data
  labels:
    app: hive
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      app: hive
  template:
    metadata:
      labels:
        app: hive
    spec:
      containers:
        - name: hive
          image: apk8s/hive-s3m:3.1.2-1.0.0
          imagePullPolicy: IfNotPresent
          env:
            - name: MYSQL_ENDPOINT
              value: mysql-mysql:3306
            - name: MYSQL_USER
              value: root
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-credentials
                  key: ROOT_PASSWORD
            - name: S3A_ENDPOINT
              value: "http://minio:9000"
            - name: S3A_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: minio-access-keys
                  key: username
            - name: S3A_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: minio-access-keys
                  key: password
          ports:
            - name: tcp-thrift-meta
              containerPort: 9083
            - name: tcp-thrift
              containerPort: 10000
```