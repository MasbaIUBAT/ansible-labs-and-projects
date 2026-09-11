FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    openssh-server \
    python3 \
    sudo \
    iproute2 \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --shell /bin/bash ansible \
    && echo "ansible ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible \
    && chmod 440 /etc/sudoers.d/ansible \
    && mkdir -p /run/sshd /home/ansible/.ssh

COPY docker/keys/ansible_lab_key.pub /home/ansible/.ssh/authorized_keys

RUN chown -R ansible:ansible /home/ansible/.ssh \
    && chmod 700 /home/ansible/.ssh \
    && chmod 600 /home/ansible/.ssh/authorized_keys

CMD ["/usr/sbin/sshd", "-D", "-e"]
