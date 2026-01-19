FROM debian:13

RUN apt-get update && \
    apt-get install -y \
    python3 python3-venv python3-pip \
    ansible \
    sshpass \
    vim \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /ansible-pack

RUN ansible-galaxy collection install community.docker

CMD ["bash"]
