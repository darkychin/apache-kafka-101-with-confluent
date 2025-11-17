FROM python:3

WORKDIR /app

COPY ./consumer.py consumer.py
COPY ./config.ini config.ini

RUN python3 -m venv env

# Need to be absolute path for the OS to know the locations of the executables 
ENV PATH="/app/env/bin:$PATH"

# Ensure python output stdout are sent straight to terminal
# https://stackoverflow.com/questions/59812009/what-is-the-use-of-pythonunbuffered-in-docker-file
ENV PYTHONUNBUFFERED=1

# Activate the environment
RUN . env/bin/activate

RUN pip install confluent-kafka

# Make the file into executable
RUN chmod u+x consumer.py

# Execute this when container is ready
CMD ["python","consumer.py","config.ini"]