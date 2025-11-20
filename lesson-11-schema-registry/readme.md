# Lesson 11: Hands-on Exercise: Confluent Schema Registry

Source of learning: https://developer.confluent.io/courses/apache-kafka/schema-registry-hands-on/

Date: 20 November 2025

We will use docker to run confluent cli. This should be straight forward if you have followed the previous tutorials to run the cli with docker.

## Prerequisite

- docker is installed and running
- follow the [lesson 11 tutorial](https://developer.confluent.io/courses/apache-kafka/schema-registry-hands-on/) until step 18

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

Use following command to consume live messages from topic "orders".

```sh
confluent kafka topic consume --from-beginning orders

# print unreadable result with weird symbols
```

As mentioned in the tutorial, the output messages are unreadable because the data is serialized using Arvo.

Run command below to tell the consumer to fetch the ARVO schema from the Schema Registry service and deserialized the data with Arvo.

```sh
confluent kafka topic consume --value-format avro --schema-registry-api-key {SCHEMA_REG_API_Key} --schema-registry-api-secret {SCHEMA_REG_API_SECRET} orders

# print readable result
```

The printed result should be readable now.

## Step 4: Stop your connector

After finish your connector usage, remember to pause your data connector to avoid unnecessary quota wastage.

Go to **Connectors** > **OrdersGenerator** > Click button "Pause"

**Congratulations!** You have completed this tutorial with docker!