FROM debian:jessie
MAINTAINER Michael J. Stealey <michael.j.stealey@gmail.com>

ENV DEBIAN_FRONTEND noninteractive

# Explicitly set user/group IDs for irods account
RUN groupadd -r irods --gid=998 \
    && useradd -r -g irods -d /var/lib/irods --uid=998 irods

# Install gosu
ENV GOSU_VERSION 1.10
RUN set -x \
    && apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
    && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
    && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true \
    && apt-get purge -y --auto-remove ca-certificates wget

RUN apt-get update && apt-get install -y \
    sudo \
    wget \
    apt-transport-https \
    libxml2

# Install iRODS v.4.2.0-preview
RUN wget -qO - https://unstable.irods.org/irods-unstable-signing-key.asc | sudo apt-key add - \
    && echo "deb [arch=amd64] https://unstable.irods.org/apt/ trusty main" | \
    sudo tee /etc/apt/sources.list.d/renci-irods-unstable.list \
    && sudo apt-get update && sudo apt-get install -y \
    irods-server

# default iRODS env
ENV IRODS_SERVICE_ACCOUNT_NAME=irods
ENV IRODS_SERVICE_ACCOUNT_GROUP=irods
# 1. provider, 2. consumer
ENV IRODS_SERVER_ROLE=2
ENV IRODS_PROVIDER_ZONE_NAME=tempZone
ENV IRODS_PROVIDER_HOST_NAME=localhost
ENV IRODS_PORT=1247
ENV IRODS_PORT_RANGE_BEGIN=20000
ENV IRODS_PORT_RANGE_END=20199
ENV IRODS_CONTROL_PLANE_PORT=1248
ENV IRODS_SCHEMA_VALIDATION=file:///var/lib/irods/configuration_schemas
ENV IRODS_SERVER_ADMINISTRATOR_USER_NAME=rods
ENV IRODS_SERVER_ZONE_KEY=TEMPORARY_zone_key
ENV IRODS_SERVER_NEGOTIATION_KEY=TEMPORARY_32byte_negotiation_key
ENV IRODS_CONTROL_PLANE_KEY=TEMPORARY__32byte_ctrl_plane_key
ENV IRODS_SERVER_ADMINISTRATOR_PASSWORD=rods
ENV IRODS_VAULT_DIRECTORY=/var/lib/irods/iRODS/Vault

# Shenanigans to make dockerhub happy
COPY . /tmp
RUN cp /tmp/docker-entrypoint.sh /docker-entrypoint.sh \
    && rm -rf /tmp/*

VOLUME /var/lib/irods/iRODS/Vault /etc/irods
ENV DEBIAN_FRONTEND teletype
EXPOSE $IRODS_PORT $IRODS_CONTROL_PLANE_PORT $IRODS_PORT_RANGE_BEGIN-$IRODS_PORT_RANGE_END
ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["setup_irods.sh"]