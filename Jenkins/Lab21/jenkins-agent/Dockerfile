FROM jenkins/inbound-agent:latest-jdk21

USER root

RUN apt-get update && \
    apt-get install -y \
    curl \
    wget \
    gnupg \
    ca-certificates \
    git \
    unzip \
    maven \
    docker.io && \
    rm -rf /var/lib/apt/lists/*

# Install Trivy
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
    | sh -s -- -b /usr/local/bin

# Install kubectl
RUN curl -fsSLo /usr/local/bin/kubectl \
    https://dl.k8s.io/release/v1.35.0/bin/linux/amd64/kubectl && \
    chmod +x /usr/local/bin/kubectl

USER jenkins