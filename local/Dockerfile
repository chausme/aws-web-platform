FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    sudo \
    curl \
 && rm -rf /var/lib/apt/lists/*

# Keep container running
CMD ["sleep", "infinity"]