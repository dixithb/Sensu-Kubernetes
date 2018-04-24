FROM oraclelinux

MAINTAINER Dixith dixithbura@gmail.com

# Basic packages
RUN yum update -y
RUN rpm -Uvh http://download.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm \
  && yum -y install passwd sudo git wget openssl openssh openssh-server openssh-clients jq

# Create user
RUN useradd sensu \
 && echo "sensu" | passwd sensu --stdin \
 && sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config \
 && sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config \
 && echo "sensu ALL=(ALL) ALL" >> /etc/sudoers.d/sensu

# Environment Variables
ENV \
  SENSU_LOG_LEVEL="info" \
  SENSU_CLIENT_SUBSCRIPTIONS="all,docker" \
  SENSU_TRANSPORT_NAME="redis" \
  REDIS_URL="redis://${SENSU_REDIS_SERVICE_HOST:-127.0.0.1}:${SENSU_REDIS_SERVICE_PORT:-6379}"

# Sensu server
ADD files/sensu.repo /etc/yum.repos.d/
RUN yum install -y sensu
RUN mkdir -p /etc/sensu/conf.d/checks
COPY files/deregistration.rb /etc/sensu/plugins/deregistration.rb
COPY files/deregistration_kubernetes.rb /etc/sensu/plugins/deregistration_kubernetes.rb
COPY files/sensu-prometheus-collector /usr/bin/sensu-prometheus-collector

RUN echo "EMBEDDED_RUBY=true" | tee /etc/default/sensu && \
RUN cd /opt/sensu/embedded/bin \
 && sensu-install -p cpu-checks
 && sensu-install -p disk-checks \
 && sensu-install -p memory-checks \
 && sensu-install -p nginx \
 && sensu-install -p process-checks \
 && sensu-install -p load-checks \
 && sensu-install -p vmstats \
 && sensu-install -p mailer \
 && sensu-install -p rest-client \
 && sensu-install -p http \
 && sensu-install -p sensu-plugins-influxdb
 && sensu-client --print_config -d /etc/sensu/conf.d/

RUN chmod +x /usr/bin/sensu-prometheus-collector
