# Use prod code-oss
FROM us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest AS cloud-ws

# we use the official Ubuntu LTS image as the base Jammy image
FROM ubuntu:22.04

# setup Cloud Workstation
RUN apt-get update && \
    apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release 

RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | \
    tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && \
    apt-get update -y
    
RUN DEBIAN_FRONTEND=noninteractive \ 
      apt-get install -y --no-install-recommends \
      bash \
      bash-completion \
      clang \
      coreutils \
      diffutils \
      dnsutils \
      dpkg \
      findutils \
      gawk \
      gcc \
      git \
      google-cloud-sdk \
      google-cloud-sdk-bigtable-emulator \
      google-cloud-sdk-cbt \
      google-cloud-sdk-datalab \
      google-cloud-sdk-datastore-emulator \
      google-cloud-sdk-gke-gcloud-auth-plugin \
      google-cloud-sdk-kpt \
      google-cloud-sdk-local-extract \
      google-cloud-sdk-minikube \
      google-cloud-sdk-skaffold \
      grep \
      gzip \
      inetutils-ping \
      kbd \
      kubectl \
      less \
      lsof \
      man-db \
      nano \
      net-tools \
      openssh-client \
      openssh-server \
      procps \
      runit \
      sudo \
      time \
      unzip \
      util-linux \
      wget \
      whois && \
    rm -rf /etc/ssh/ssh_host_*

# Install docker
RUN mkdir -p /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN cat /etc/apt/keyrings/docker.gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null && \
	 apt-get update && \
    apt-get install -y docker-ce docker-ce-cli && \
    apt-get clean && \
    rm -rf /apt-keys

# Create volume for docker-in-docker overlay
VOLUME /var/lib/docker
# Update the docker data directory to use the home dir so its persisted
# between sessions.
ENV DOCKER_OPTS="-g /home/.docker_data"

COPY --from=cloud-ws /opt/code-oss /opt/code-oss
COPY --from=cloud-ws /etc/workstation-startup.d /etc/workstation-startup.d
COPY --from=cloud-ws /google /google
COPY --from=cloud-ws /etc/profile.d/disable_gcloud_gce_check.sh /etc/profile.d/disable_gcloud_gce_check.sh
COPY --from=cloud-ws /etc/profile.d/go_envs.sh /etc/profile.d/go_envs.sh
COPY --from=cloud-ws /etc/profile.d/minikube_envs.sh /etc/profile.d/minikube_envs.sh
COPY --from=cloud-ws /etc/ssh/sshd_config /etc/ssh/sshd_config
COPY --from=cloud-ws /etc/workstation-startup.d/000_configure-docker.sh /etc/workstation-startup.d/000_configure-docker.sh
COPY --from=cloud-ws /etc/workstation-startup.d/010_add-user.sh /etc/workstation-startup.d/010_add-user.sh
COPY --from=cloud-ws /etc/workstation-startup.d/020_start-sshd.sh /etc/workstation-startup.d/020_start-sshd.sh
COPY --from=cloud-ws /usr/bin/workstation-startup /usr/bin/workstation-startup
COPY --from=cloud-ws /etc/bash.bashrc /etc/bash.bashrc

# Install packages for needed development
# Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
RUN sudo apt update -y
RUN sudo apt install nodejs -y

# Hugo
RUN sudo apt install -y hugo libsass1

# npm packages to install globally
RUN npm install -g firebase-tools

ENTRYPOINT ["/google/scripts/entrypoint.sh"]