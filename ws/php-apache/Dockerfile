FROM us-central1-docker.pkg.dev/cloud-workstations-images/predefined/code-oss:latest AS cloud-ws

# Ideally would go with a Workstations Base Image, but Apache PHP looks like a pain
# so lets use that as a base image and copy from the workstation image
FROM php:7.4-apache

COPY 000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ports.conf /etc/apache2/ports.conf

# Set ServerName to localhost to remove unnecessary warnings
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf && \
    a2enmod rewrite && \
    a2enmod userdir && \
    service apache2 restart

#TODO: Install composer and PHP extensions

# add public_html to skeleton so will automatically create dir for each user
RUN mkdir /etc/skel/public_html
RUN echo "<h1>Working directory</h1>" > /etc/skel/public_html/index.html

# setup Cloud Workstation
RUN apt-get update && \
    apt-get install -y apt-transport-https ca-certificates gnupg

RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | \
    tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
    apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && \
    apt-get update -y

RUN apt-get install -y --no-install-recommends \
      bash \
      bash-completion \
      clang \
      coreutils \
      curl \
      diffutils \
      dnsutils \
      dpkg \
      findutils \
      gawk \
      gcc \
      git \
      gnupg \
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
      supervisor \
      time \
      unzip \
      util-linux \
      wget \
      whois && \
    rm -rf /etc/ssh/ssh_host_*

# install Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
RUN sudo apt update -y

RUN sudo apt install nodejs -y
RUN echo "Node: " && node -v
RUN echo "NPM: " && npm -v

ENV PATH=$PATH:/usr/local/go/bin
RUN wget https://storage.googleapis.com/golang/go1.19.linux-amd64.tar.gz && \
    echo 464b6b66591f6cf055bc5df90a9750bf5fbc9d038722bb84a9d56a2bea974be6 go1.19.linux-amd64.tar.gz | sha256sum -c - && \
    tar xzf go1.19.linux-amd64.tar.gz -C /usr/local && \
    rm go1.19.linux-amd64.tar.gz

# Install docker
RUN mkdir -p /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN cat /etc/apt/keyrings/docker.gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo \
       "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
        $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
	 apt-get update && \
    apt-get install -y docker-ce docker-ce-cli && \
    apt-get clean && \
    rm -rf /apt-keys

COPY --from=cloud-ws /opt/code-oss /opt/code-oss
COPY --from=cloud-ws /etc/workstation-startup.d /etc/workstation-startup.d
COPY --from=cloud-ws /google /google

# Install kind
RUN export GOPATH=/google/gopath && \
  /usr/local/go/bin/go install sigs.k8s.io/kind@v0.16.0
ENV PATH=$PATH:/google/gopath/bin

# Create volume for docker-in-docker overlay
VOLUME /var/lib/docker

# Update the docker data directory to use the home dir so its persisted
# between sessions.
ENV DOCKER_OPTS="-g /home/.docker_data"

COPY --from=cloud-ws /etc/profile.d/default_envs.sh /etc/profile.d/default_envs.sh
COPY --from=cloud-ws /etc/profile.d/disable_gcloud_gce_check.sh /etc/profile.d/disable_gcloud_gce_check.sh
COPY --from=cloud-ws /etc/profile.d/go_envs.sh /etc/profile.d/go_envs.sh
COPY --from=cloud-ws /etc/profile.d/minikube_envs.sh /etc/profile.d/minikube_envs.sh

COPY --from=cloud-ws /etc/ssh/sshd_config /etc/ssh/sshd_config

COPY --from=cloud-ws /etc/workstation-startup.d/000_configure-docker.sh /etc/workstation-startup.d/000_configure-docker.sh
COPY --from=cloud-ws /etc/workstation-startup.d/010_add-user.sh /etc/workstation-startup.d/010_add-user.sh
COPY --from=cloud-ws /etc/workstation-startup.d/020_start-sshd.sh /etc/workstation-startup.d/020_start-sshd.sh

COPY --from=cloud-ws /usr/bin/workstation-startup /usr/bin/workstation-startup

COPY --from=cloud-ws /etc/bash.bashrc /etc/bash.bashrc

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord"]

