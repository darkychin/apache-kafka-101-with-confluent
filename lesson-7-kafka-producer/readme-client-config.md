# Client Config file (Nodejs)

We will create a client config file in **nodejs** (because I understand **nodejs**).

You can find config file for different language [here](https://docs.confluent.io/confluent-cli/current/command-reference/kafka/client-config/create/index.html).
 
## Method 1: Printing client config
To print the client config file content, run command below in **confluent-cli** after you have selected your **environment** and **cluster**, or you can just pass them in directly in the command.
```sh
# in shell
# with preselection
confluent kafka client-config create nodejs --api-key <API_KEY> --api-secret <API_SECRET>

# pass everything as arguments
confluent kafka client-config create nodejs --environment <ENV_ID> --cluster <CLUSTER_ID> --api-key <API_KEY> --api-secret <API_SECRET>


# configs content result
# ....
```
[Source](https://docs.confluent.io/confluent-cli/current/command-reference/kafka/client-config/create/confluent_kafka_client-config_create_nodejs.html)


## Method 2: Generate a readable `.config` file

## Problem Statement
To create a readable `.config` file, the file must be stored in the mounted volume of host directory.

When using the original **confluent-cli** image, we will face "permission denied" problem with mounted host directory, because the confluent-cli image create its own user, and it does not inherit the users from docker host.

[This is a very common docker permission problem.](https://stackoverflow.com/questions/39397548/how-to-give-non-root-user-in-docker-container-access-to-a-volume-mounted-on-the)

The best way to do this is to build a new image extending from confluent-cli's image, with new user with the same id as the docker host.

> Side note:  
> You can set user as root during `docker run` with confluent-cli, but after tutorial complete, you are required to `sudo` in order to remove the created file, which is cumbersome.
>
> Futhermore, providing root access to a program is dangerous.
> ```sh
> docker run \
> -it \
> -v ./config:/config \  # mount volume to generate config file to host's directory ./config
> -u 0 \  # start user as root
> --name confluent-cli \
> confluentinc/confluent-cli:4.43.0 sh
> ```

## Solution
### Step 1: Create config directory
Create a directory "/config" for our config.

We need it because `docker run` with mounted volume will [create the directory for us in root access when the directory is missing](
https://stackoverflow.com/q/39794793/7939633).

Doing this avoid the need of `sudo`.

### Step 2: Create **Dockerfile**
Create **Dockerfile** with content below ([find it here](./Dockerfile))
```dockerfile
# Dockerfile

# extend from confluent-cli
FROM confluentinc/confluent-cli:4.43.0

# Switch to root temporarily to add new user to group
USER root

# Create a new user matching your host (replace 1000 with your actual UID/GID in build command)
ARG HOST_UID=1000

# the host's user name in this container
ENV HOST_NAME=hostuser

# add new userId to the group `confluent`
# $USER = "confluent" (group name & user name from confluent-cli image)
RUN adduser -D -u $HOST_UID -G $USER $HOST_NAME

# Update files permission to user "hostuser" to ensure the confluent-cli binary is executable by the new host user
RUN chown $HOST_NAME /etc
RUN chown $HOST_NAME /bin

# Switch to the new user
USER hostuser
```

### Step 3: Build the image
Run build command below to:
1. build the image with current docker host user id as argument `HOST_UID`
2. label the image with name "my-confluent-cli" with tag "latest" 
```sh
docker build --build-arg HOST_UID=$(id -u) -t my-confluent-cli:latest .
```

### Step 4: Run the container
Run command to:
1. Create a container with image "my-confluent-cli"
2. Use flag `-it` to turn on interactive mode
3. Mount the shared volume directory "/config"
4. Named the container as "custom-confluent-cli"
5. Open the container's shell
```sh
# in directory with "/config"
docker run \                                                                 
-it \
-v ./config:/config \
--name custom-confluent-cli \
my-confluent-cli sh
```

### Step 5: Login confluent-cli
```sh
# in shell
confluent login --save
# complete login
```

### Step 6: Generate client config file
Run command below after you have selected your **environment** and **cluster**, or you can just pass them in directly in the command.

This command will generate "my-client-config-file.config" under host directory "/config" (config file example can be found [here](./config/my-client-config-file%20copy.config.sample)).

If any error occurred, it will log them in the terminal.
```sh
# with preselection
confluent kafka client-config create nodejs --api-key <API_KEY> --api-secret <API_SECRET> 1> /config/my-client-config-file.config 2>&1

# pass everything as arguments
confluent kafka client-config create nodejs --environment <ENV_ID> --cluster <CLUSTER_ID> --api-key <API_KEY> --api-secret <API_SECRET>  1> /config/my-client-config-file.config 2>&1
```
[Source](https://docs.confluent.io/confluent-cli/current/command-reference/kafka/client-config/create/confluent_kafka_client-config_create_nodejs.html)


Credits:
- https://stackoverflow.com/questions/39397548/how-to-give-non-root-user-in-docker-container-access-to-a-volume-mounted-on-the
- GrokAI: 
    - suggestions to solve docker volume permission denied across different users
    - To view the untruncated, full commands (including all arguments and variable references as written in the original Dockerfile), pull the image locally and use:  
      ```sh
      docker pull confluentinc/confluent-cli:4.42.0
      docker history --no-trunc --format "{{.CreatedBy}}" $(docker images -q confluentinc/confluent-cli:4.43.0)
      ```
