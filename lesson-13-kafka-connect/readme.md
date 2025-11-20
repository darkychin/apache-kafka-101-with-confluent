# Lesson 13: Hands-on Exercise: Kafka Connect

Source of learning: https://developer.confluent.io/courses/apache-kafka/exercise-kafka-connect/

Date: 20 November 2025

We will use docker to run confluent cli. 

This lesson actually is identical with [lesson 11](../lesson-11-schema-registry/readme.md), but without choosing schema format that require serialization and deserialization such as ARVO and this tutorial uses different data source.

But as a completionist, I will created the steps regardless (prepare for it to sound like a broken record).

## Prerequisite

- docker is installed and running
- follow the [lesson 13 tutorial](https://developer.confluent.io/courses/apache-kafka/exercise-kafka-connect/) until step 10

## Step 1: Run confluent cli in docker

Start previous confluent-cli or create a new container in docker and open shell.

```sh
# run previous container
docker start confluent-cli && docker exec -it confluent-cli sh

# or

# create a new container
docker run \
-it \
--name confluent-cli \
confluentinc/confluent-cli:4.43.0 sh
```

## Step 2: Login and select environment and cluster

Login confluent cloud, then select the targeted environment and cluster.

```sh
# login
confluent login --save

# select environment & cluster
confluent environment use <ENV_ID> && confluent kafka cluster use <CLUSTER_ID>
```

## Step 3: Consume data

Use following command to consume live messages from topic "inventory".

```sh
confluent kafka topic consume --from-beginning inventory

# print results in json
```

**Congratulations!** You have completed this tutorial with docker!