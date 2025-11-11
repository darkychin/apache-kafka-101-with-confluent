# Extra Knowledge

# Docker

## 1. **Dockerfile** `RUN` command with `/bin/sh -c`

Please do not write `RUN /bin/sh -c` in **Dockerfile** because docker default command for `RUN` is `/bin/sh -c`, and it will end up with `RUN /bin/sh -c /bin/sh -c`.

Only add your targeted command when it is not using `/bin/sh -c`.

Example

```dockerfile
# Dockerfile
# ...

# execution success
RUN /bin/echo $(cat /etc/group)   # print all group in the system

# execution failed with error, `failed to solve: process "/bin/sh -c /bin/sh -c chown $HOST_NAME /etc"`
RUN /bin/sh -c chown $HOST_NAME /etc   # add permission for $HOST_NAME to access `/etc`

# execution success
RUN chown $HOST_NAME /etc   # add permission for $HOST_NAME to access `/etc`

# ...
```

Credit: https://stackoverflow.com/a/39777387/7939633

## 2. **Dockerfile** `RUN` vs `CMD`

`RUN` is an image build step, the state of the container after a RUN command will be committed to the container image. A Dockerfile can have many RUN steps that layer on top of one another to build the image.

`CMD` is the command the container executes by default when you launch the built image. A Dockerfile will only use the final CMD defined. The CMD can be overridden when starting a container with docker run $image $other_command.

[Source](https://stackoverflow.com/a/37462208/7939633)

## 3. In **Dockerfile**, how to `echo`(print) a system command?

Example:

```dockerfile
# Dockerfile
RUN /bin/echo $(cat /etc/group) # Printing groups in system
RUN /bin/echo $(cat /etc/passwd)  # Printing users in system
```

Credits

- GrokAI (verified)

## 4. Docker building an image with root

As default, Docker will always build an image starting with **root** access.

Hence, in best practice, custom image should always create a **group** and a **non-root user** and switch to it after the environment is completely set up.

Example:

```dockerfile
# set up all environment in alpine busybox
# ...

ENV GID=2358
ENV UID=2358
ENV GROUP=confluentgroup
ENV USER=confluent

# create a new system group
RUN addgroup -G $GID -S $GROUP

# add user to the group
RUN adduser -D -u $UID -G $GROUP $USER

# switch to user
USER confluent
```

Code reference

- https://hub.docker.com/layers/confluentinc/confluent-cli/latest/images/sha256-b8e7ae597bd623e390f2229df1ff17c322b065ea46560e97344d27adb2e6a0ed
- docker history --no-trunc --format "{{.CreatedBy}}" $(docker images -q confluentinc/confluent-cli:4.43.0)
- https://busybox.net/BusyBox.html

## 5. Reading complete docker image vs docker image on DockerHub

Question:  
Why docker image in **DockerHub** is missing usage of their `ENV` variables and how to print the raw file?

Answer:  
This happens due to image on **DockerHub** is truncated.

[DockerHub image with truncation](https://hub.docker.com/layers/confluentinc/confluent-cli/latest/images/sha256-b8e7ae597bd623e390f2229df1ff17c322b065ea46560e97344d27adb2e6a0ed)

```dockerfile
ADD alpine-minirootfs-3.22.2-x86_64.tar.gz / # buildkit

CMD ["/bin/sh"]

ENV USER=confluent

ENV UID=2358

ENV GID=2358

RUN /bin/sh -c addgroup

RUN /bin/sh -c adduser

RUN /bin/sh -c apk update

COPY confluent /bin # buildkit

RUN /bin/sh -c chown $USER:$USER

RUN /bin/sh -c chown $USER:$USER

RUN /bin/sh -c ln -s

RUN /bin/sh -c mkdir -p

USER confluent
```

Complete file from command  
`docker history --no-trunc --format "{{.CreatedBy}}" $(docker images -q confluentinc/confluent-cli:4.43.0)`

```dockerfile
# read from bottom to up
USER confluent
RUN /bin/sh -c mkdir -p /etc/bash_completion.d/ && confluent completion bash > /etc/bash_completion.d/confluent # buildkit
RUN /bin/sh -c ln -s /bin/confluent /confluent # buildkit
RUN /bin/sh -c chown $USER:$USER /etc # buildkit
RUN /bin/sh -c chown $USER:$USER /bin # buildkit
COPY confluent /bin # buildkit
RUN /bin/sh -c apk update --no-cache &&     apk add --no-cache jq &&     apk add --no-cache curl &&     apk add --no-cache bash &&     apk add --no-cache bash-completion # buildkit
RUN /bin/sh -c adduser     --disabled-password     --gecos ""     --ingroup "$USER"     --uid "$UID"     "$USER" # buildkit
RUN /bin/sh -c addgroup     --gid $GID     --system     $USER # buildkit
ENV GID=2358
ENV UID=2358
ENV USER=confluent
CMD ["/bin/sh"]
ADD alpine-minirootfs-3.22.2-x86_64.tar.gz / # buildkit
```

Credit

- GrokAI (verified)

## 5. Run dockerbuild with printed error and show max verbose

```sh
docker build --progress=plain --no-cache ... .
```

Source

- https://stackoverflow.com/a/67682576/7939633
- GrokAI (verified)

## 6. User id permission are inherited from Docker host to Docker container

When directory eg: `/test` is created by host user, the user with the same id in the container can inherit the same permission.

Examples:  
Base docker run with mount

```sh
docker run \
-v ./test:/test \
# ...
```

### Situation 1

Host User Id: 1000  
Container User Id: 1000

In container:

1. Create a user with id 1000
2. Switch to user id 1000
3. Add a file into `/test`
4. Success

Container user allow to read and write to `/test`

### Situation 2

Host User Id: 1000  
Container User Id: 2358

In container:

1. Create a user with id 2358
2. Switch to user id 2358
3. Add a file into `/test`
4. Failed! Permission denied.

Container user is not allow to read or write to `/test`

# Linux - busybox

## 1. command `adduser`

- https://www.linuxquestions.org/questions/linux-newbie-8/userlist-in-busybox-711726/
- https://wiki.alpinelinux.org/wiki/Setting_up_a_new_user
- https://busybox.net/BusyBox.html
- [flags](https://stackoverflow.com/a/49955098/7939633)

## 2. Printing groups in system

```sh
cat /etc/group
```

## 3. Printing users in system

```sh
cat /etc/passwd
```

## 4. Check who is the current user

```sh
whoami
```

## 5. Check permission level of a directory/file

```sh
ls -ld <pathToDirectory/File>
```
