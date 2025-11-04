# For Generating the readable config file

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


# References:
# official file - https://hub.docker.com/layers/confluentinc/confluent-cli/latest/images/sha256-b8e7ae597bd623e390f2229df1ff17c322b065ea46560e97344d27adb2e6a0ed
# docker command to read the original untrimmed confluent cli image - `docker history --no-trunc --format "{{.CreatedBy}}" $(docker images -q confluentinc/confluent-cli:4.43.0)`

# Credit on issues and its solution:
# - why docker image in docker hub is missing usage of their ENV variables and how to print the raw file (Guided by Grok AI)
# - how to run `docker build` that allow `echo` and shows details build error
#     - https://stackoverflow.com/questions/64804749/why-is-docker-build-not-showing-any-output-from-commands/67682576#67682576
#     - Grok AI 
# - do not add "/bin/sh -c" in dockerfile `RUN` (https://stackoverflow.com/a/39777387/7939633)