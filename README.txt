## Docker-scanner
## Stage 1: Fetching necessary binaries
FROM debian:stable-slim AS fetcher

## Install curl and other necessary tools in the fetcher stage
RUN apt-get update && apt-get install -y curl

## Fetching binaries
WORKDIR /tmp

# Download ctop (v0.7.7)
RUN curl -LO https://github.com/bcicen/ctop/releases/download/v0.7.7/ctop-0.7.7-linux-amd64 \
    && chmod +x ctop-0.7.7-linux-amd64 \
    && mv ctop-0.7.7-linux-amd64 /usr/local/bin/ctop

# Download calicoctl (v3.26.5)
RUN curl -LO https://github.com/projectcalico/calico/releases/download/v3.26.5/calicoctl-linux-amd64 \
    && chmod +x calicoctl-linux-amd64 \
    && mv calicoctl-linux-amd64 /usr/local/bin/calicoctl

# Download and extract termshark (v2.4.0)
RUN curl -LO https://github.com/gcla/termshark/releases/download/v2.4.0/termshark_2.4.0_linux_x64.tar.gz \
    && tar -xzf termshark_2.4.0_linux_x64.tar.gz \
    && mv termshark_2.4.0_linux_x64/termshark /usr/local/bin/termshark

# Download and extract grpcurl (v1.8.7)
RUN curl -LO https://github.com/fullstorydev/grpcurl/releases/download/v1.8.7/grpcurl_1.8.7_linux_x86_64.tar.gz \
    && tar -xzf grpcurl_1.8.7_linux_x86_64.tar.gz \
    && mv grpcurl /usr/local/bin/grpcurl

# Download fortio (v1.67.1) and extract
RUN curl -LO https://github.com/fortio/fortio/releases/download/v1.67.1/fortio-linux_amd64-1.67.1.tgz \
    && tar -xzf fortio-linux_amd64-1.67.1.tgz \
    && mv usr/bin/fortio /usr/local/bin/fortio

# Fix permissions for all binaries
RUN chmod +x /usr/local/bin/ctop /usr/local/bin/calicoctl /usr/local/bin/termshark /usr/local/bin/grpcurl /usr/local/bin/fortio

# Stage 2: Building the final image
FROM alpine:3.18.6

# Use stable repositories
RUN set -ex \
    && echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/main" >> /etc/apk/repositories \
    && echo "http://dl-cdn.alpinelinux.org/alpine/v3.18/community" >> /etc/apk/repositories \
    && apk update \
    && apk upgrade \
    && apk add --no-cache \
    apache2-utils \
    bash \
    bind-tools \
    bird \
    bridge-utils \
    busybox-extras \
    conntrack-tools \
    curl \
    dhcping \
    drill \
    ethtool \
    file \
    fping \
    iftop \
    iperf \
    iperf3 \
    iproute2 \
    ipset \
    iptables \
    iptraf-ng \
    iputils \
    ipvsadm \
    httpie \
    jq \
    libc6-compat \
    liboping \
    ltrace \
    mtr \
    net-snmp-tools \
    netcat-openbsd \
    nftables \
    ngrep \
    nmap \
    nmap-nping \
    openssl \
    py3-pip \
    py3-setuptools \
    scapy \
    socat \
    speedtest-cli \
    openssh \
    zsh \
    strace \
    tcpdump \
    tcptraceroute \
    tshark \
    util-linux \
    vim \
    git \
    websocat \
    perl-crypt-ssleay \
    perl-net-ssleay

# Installing oh-my-zsh
RUN git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh

# Ensure binaries are available in the final image
COPY --from=fetcher /usr/local/bin/ctop /usr/local/bin/ctop
COPY --from=fetcher /usr/local/bin/calicoctl /usr/local/bin/calicoctl
COPY --from=fetcher /usr/local/bin/termshark /usr/local/bin/termshark
COPY --from=fetcher /usr/local/bin/grpcurl /usr/local/bin/grpcurl
COPY --from=fetcher /usr/local/bin/fortio /usr/local/bin/fortio

# Setting user and home directory
USER root
WORKDIR /root
ENV HOSTNAME=netshoot

# Copy ZSH configuration and Message of the Day
COPY zshrc .zshrc
COPY motd motd

# Fix permissions for OpenShift and tshark
RUN chmod -R g=u /root
RUN chown root:root /usr/bin/dumpcap

# Running ZSH shell by default
CMD ["zsh"]
