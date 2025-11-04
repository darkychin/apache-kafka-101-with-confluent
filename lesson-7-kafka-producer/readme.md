# Lesson 7: Hands-on Exercise: Kafka Producer  
Source of learning: https://developer.confluent.io/courses/apache-kafka/get-started-hands-on/

Starting from section **"Set Up the Confluent CLI"**.

I will be using docker to complete this tutorial for reasons:
- keep my environment clean
- hands on for my docker knowledge 

## Prerequsite
- docker is installed and running

## Step 1: Run docker command
Run following docker command to:
1. Pull the Confluent CLI image version v4.43.0
2. Run the container with that image
3. Use flag `-it` to turn on interactive mode
4. Name the container as `confluent-cli`
5. Open the container's `shell`

```sh
# current directory: apache-kafka-101-with-confluent/lesson-7-kafka-producer 
docker run \
-it \
--name confluent-cli \
confluentinc/confluent-cli:4.43.0 sh
```

There is no `--rm` flag in the `docker run` command to ensure that the container configs including credentials are not removed when we stop the container.

> Note:  
> To restart your container after exit, run  
> `docker start confluent-cli`  
>
> To stop your container, run  
> `docker stop confluent-cli`

## Step 2: Check confluent-cli
Run following command to ensure that your confluent-cli is running
```sh
confluent version

# print result with version
# confluent - Confluent CLI

# Version:     v4.43.0
# Git Ref:     14b42e02
# Build Date:  2025-10-24T21:11:53Z
# Go Version:  go1.24.6 X:boringcrypto (linux/amd64)
# Development: false
```

## Step 3: Follow the tutorial instructions
Follow the remaining instructions from section [**Set Up the Confluent CLI**](https://developer.confluent.io/courses/apache-kafka/get-started-hands-on/#set-up-the-confluent-cli) till the end.

## Set Up the Confluent CLI 

### 1. Login
```sh
confluent login --save
```
Flag `--save` here will store your credential to a local file in the container and reduce your repeated logins.  


### 2. Select your environment and cluster
```sh
# select your environment
# run `confluent environment list` to list your available environment(s)
confluent environment use <Env_Id>

# select your cluster
# run `confluent kafka cluster list` to list your available cluster(s)
confluent kafka cluster use <Cluster_Id>
```

### 3. Use an API key and secret in the CLI
1. 
    a. Add existing API key to the CLI with command
    ```sh
    confluent api-key store --resource <Cluster_Id> <API_Key> <API_Secret>
    ```
    b. Or create a new key with the command 
    ```sh
    confluent api-key create --resource <Cluster_Id>
    ```

2. Then use the API key with command
```sh
confluent api=key use <API_Key>
```

3. [optional] Click [here](./readme-client-config.md) to learn how to create a client config.

## Produce and Consume Using the Confluent CLI

### 1. List available topic list
```sh
confluent kafka topic list
```
`thermostat_readings` should exist in the printed result.

### 2. Consume messages from the beginning
Run following command to become the consumer
```sh
confluent kafka topic consume --from-beginning thermostat_readings
```
The `--from-beginning` flag tells the consumer to start from the earliest known offset on the topic, i.e., the earliest message. Leave this terminal window open to observe new messages.  
(copy directly from [source](https://developer.confluent.io/courses/apache-kafka/get-started-hands-on/#set-up-the-confluent-cli) )

### 3. Open a new terminal to produce more messages
Open another terminal to send the message to the consumer.

Run this command to connect to the running container `confluent-cli`
```sh
docker exec -it confluent-cli sh
```

Then run your producer command then paste the following data
```sh
confluent kafka topic produce thermostat_readings --parse-key

# when prompted paste the following data
299:{"sensor_id":299,"location":"living room","temperature":21,"read_at":1736522046}
151:{"sensor_id":151,"location":"bedroom","temperature":21,"read_at":1736522043}
42:{"sensor_id":42,"location":"kitchen","temperature":26,"read_at":1736522101}
```

### 4. Switch back to the terminal with consumer
Open your terminal with consumer to see the messages are received.

### 5. Check the new messages via Confluent Cloud UI
You can also check out Confluent Cloud UI to see the messages:

Go to **Environments** > **default** > **your-cluster-name** > **Topics** > **"thermostat_readings"** > **Messages**

**Congratulations!** You have completed this tutorial with docker!