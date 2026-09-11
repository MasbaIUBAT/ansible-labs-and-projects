FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    ansible-core \
    openssh-client \
    python3 \
    iputils-ping \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p /root/.ssh

COPY docker/keys/ansible_lab_key /root/.ssh/ansible_lab_key

RUN chmod 600 /root/.ssh/ansible_lab_key

WORKDIR /workspace

CMD ["sleep", "infinity"]
