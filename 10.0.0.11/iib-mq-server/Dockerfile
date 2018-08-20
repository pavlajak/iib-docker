FROM rhel7 

LABEL maintainer "Jakub Pavlat <jakub_pavlat@cz.ibm.com>"

LABEL "ProductID"="447aefb5fd1342d5b893f3934dfded73" \
      "ProductName"="IBM Integration Bus" \
      "ProductVersion"="10.0.0.11"

# The URL to download the MQ installer from in tar.gz format
ARG MQ_URL=https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/mqadv_dev910_linux_x86-64.tar.gz
ARG IIB_URL=http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/integration/10.0.0.11-IIB-LINUX64-DEVELOPER.tar.gz
ARG MQ_FN=mqadv_dev910_linux_x86-64.tar.gz
ARG IIB_FN=10.0.0.11-IIB-LINUX64-DEVELOPER.tar.gz

# Install additional packages required by MQ and IIB, this install process and the runtime scripts
RUN yum install -y -q \
    bash \
    bc \
    ca-certificates \
    coreutils \
    curl \
    file \
    findutils \
    gawk \
    grep \
    libc-bin \
    lsb-release \
    mount \
    passwd \
    procps \
    rsyslog \
    sed \
    sudo \
    tar \
    util-linux

# Download and extract the MQ installation files
RUN export DIR_EXTRACT=/tmp/mq \
  && mkdir -p ${DIR_EXTRACT} \ 
  && cd ${DIR_EXTRACT}


# Download MQ installation files
COPY ${MQ_FN} . 
### RUN curl -LO $MQ_URL \

# Unpack archive
RUN tar -zxvf ./${MQ_FN}

# Recommended: Create the mqm user ID with a fixed UID and group, so that the file permissions work between different images
RUN groupadd --system --gid 1011 mqm \
  && useradd --system --uid 1011 --gid mqm mqm \
  && usermod -G mqm root

# Install MQ using rpm
RUN MQServer/mqlicense.sh -text_only -accept \
  && rpm -ivh ${DIR_EXTRACT}/MQServer/MQSeries*.rpm \

  # Clean up all the downloaded files
  && rm -rf ${DIR_EXTRACT}

ARG IIB_URL=http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/integration/10.0.0.11-IIB-LINUX64-DEVELOPER.tar.gz


COPY 10.0.0.11-IIB-LINUX64-DEVELOPER.tar.gz .
# Install IIB V10 Developer edition
RUN mkdir /opt/ibm && \
###    curl -LO $IIB_URL \ 
    && tar -zxvf --exclude iib-10.0.0.11/tools --directory /opt/ibm \
    && /opt/ibm/iib-10.0.0.11/iib make registry global accept license silently

# Create user to run as
RUN groupadd -f mqbrkrs && \
    groupadd -f mqclient && \
    useradd --create-home --home-dir /home/iibuser -G mqbrkrs,sudo,mqm,mqclient iibuser && \
    sed -e 's/^%sudo	.*/%sudo	ALL=NOPASSWD:ALL/g' -i /etc/sudoers

# Copy in script files
COPY *.sh /usr/local/bin/
COPY mq-config /etc/mqm/mq-config
RUN chmod 755 /usr/local/bin/*.sh \
  && chmod 755 /etc/mqm/mq-config

# Set BASH_ENV to source mqsiprofile when using docker exec bash -c
ENV BASH_ENV=/usr/local/bin/iib_env.sh MQSI_MQTT_LOCAL_HOSTNAME=127.0.0.1 MQSI_DONT_RUN_LISTENER=true LANG=en_US.UTF-8

# Expose default admin port and http ports
EXPOSE 4414 7800 1414

USER iibuser

# Set entrypoint to run management script
ENTRYPOINT ["iib_manage.sh"]
