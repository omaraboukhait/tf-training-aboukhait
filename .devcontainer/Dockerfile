FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# ── System packages ──────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    sudo \
    curl \
    wget \
    unzip \
    git \
    jq \
    ca-certificates \
    gnupg \
    vim \
    nano \
    less \
    tree \
    bash-completion \
    && rm -rf /var/lib/apt/lists/*

# ── Terraform 1.15.5 ─────────────────────────────────────────────────────────
ARG TERRAFORM_VERSION=1.15.5
RUN wget -q -O /tmp/terraform.zip \
    "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" \
    && unzip /tmp/terraform.zip -d /usr/local/bin \
    && rm /tmp/terraform.zip \
    && terraform version

# ── AWS CLI v2 ───────────────────────────────────────────────────────────────
RUN curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o /tmp/awscliv2.zip \
    && unzip /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
    && rm -rf /tmp/aws /tmp/awscliv2.zip \
    && aws --version

# ── Terraform tab-completion ─────────────────────────────────────────────────
RUN terraform -install-autocomplete || true

WORKDIR /workspaces
