FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    curl \
    default-mysql-client \
    python3 \
    python3-pip \
    sudo \
    vim \
 && rm -rf /var/lib/apt/lists/*

# Keep container running
CMD ["sleep", "infinity"]