# syntax=docker/dockerfile:1
ARG MAJOR_VERSION=6.4
ARG RELEASE=20
ARG ZBX_VERSION=${MAJOR_VERSION}.20
ARG BUILD_BASE_IMAGE=zabbix-build-pgsql:rhel-${ZBX_VERSION}

FROM ${BUILD_BASE_IMAGE} AS builder

FROM registry.access.redhat.com/ubi9/ubi-minimal:9.5

ARG MAJOR_VERSION
ARG RELEASE
ARG ZBX_VERSION
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES} \
    MIBDIRS=/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL \
    NMAP_PRIVILEGED=""

LABEL description="Zabbix server with PostgreSQL database support" \
      maintainer="alexey.pustovalov@zabbix.com" \
      name="zabbix/zabbix-server-pgsql-64" \
      release="${RELEASE}" \
      run="docker run --name zabbix-server --link postgresql:pgsql-server -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-server-pgsql-64:${ZBX_VERSION}" \
      summary="Zabbix server (PostgreSQL)" \
      url="https://www.zabbix.com/" \
      vendor="Zabbix SIA" \
      version="${MAJOR_VERSION}" \
      io.k8s.description="Zabbix server with PostgreSQL database support" \
      io.k8s.display-name="Zabbix server (PostgreSQL)" \
      io.openshift.expose-services="10051:10051" \
      io.openshift.tags="zabbix,zabbix-server,pgsql" \
      org.label-schema.build-date="${BUILD_DATE}" \
      org.label-schema.description="Zabbix server with PostgreSQL database support" \
      org.label-schema.docker.cmd="docker run --name zabbix-server --link postgresql:pgsql-server -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-server-pgsql-64:${ZBX_VERSION}" \
      org.label-schema.license="GPL v2.0" \
      org.label-schema.name="zabbix-server-pgsql-rhel" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.url="https://zabbix.com/" \
      org.label-schema.usage="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.label-schema.vcs-ref="${VCS_REF}" \
      org.label-schema.vcs-url="${ZBX_SOURCES}" \
      org.label-schema.vendor="Zabbix SIA" \
      org.label-schema.version="${ZBX_VERSION}"

STOPSIGNAL SIGTERM

COPY ["licenses", "/licenses"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}-output/server/sbin/zabbix_server", "/usr/sbin/zabbix_server"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}-output/general/bin/*", "/usr/bin/"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}-output/server/conf/", "/etc/zabbix/"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}-output/server/database/postgresql/", "/usr/share/doc/zabbix-server-postgresql/"]

RUN --mount=type=tmpfs,target=/var/lib/dnf/ \
    --mount=type=bind,target=/run/secrets/,src=secrets/ \
    set -eux && \
    INSTALL_PKGS="bash \
            traceroute \
            nmap \
            fping \
            shadow-utils \
            iputils \
            hostname \
            libssh \
            libpsl \
            libbrotli \
            libevent \
            openldap \
            libssh \
            libxml2 \
            net-snmp-agent-libs \
            OpenIPMI-libs \
            pcre2 \
            postgresql \
            postgresql-private-libs \
            gzip \
            unixODBC \
            zlib" && \
    curl --tlsv1.2 -sSf -L https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm -o /tmp/epel-release-latest-9.noarch.rpm && \
    rpm -ivh /tmp/epel-release-latest-9.noarch.rpm && \
    rm -rf /tmp/epel-release-latest-9.noarch.rpm && \
    ARCH_SUFFIX="$(arch)"; \
    microdnf -y module enable \
        --disablerepo "*" \
        --enablerepo "rhel-9-for-$ARCH_SUFFIX-appstream-rpms" \
        --setopt=install_weak_deps=0 \
        --setopt=keepcache=0 \
        postgresql:16 && \
    microdnf -y install \
            --disablerepo "*" \
            --enablerepo "ubi-9-baseos-rpms" \
            --enablerepo "ubi-9-appstream-rpms" \
            --enablerepo "rhel-9-for-$ARCH_SUFFIX-baseos-rpms" \
            --enablerepo "rhel-9-for-$ARCH_SUFFIX-appstream-rpms" \
            --enablerepo "codeready-builder-for-rhel-9-$ARCH_SUFFIX-rpms" \
            --enablerepo "epel" \
            --setopt=install_weak_deps=0 \
            --setopt=keepcache=0 \
            --best \
            --setopt=tsflags=nodocs \
        ${INSTALL_PKGS} && \
    microdnf -y update \
            --disableplugin=subscription-manager \
            --disablerepo "*" \
            --enablerepo "ubi-9-baseos-rpms" \
            --setopt=install_weak_deps=0 \
            --best \
            --setopt=tsflags=nodocs \
        tzdata && \
    microdnf -y reinstall \
            --disableplugin=subscription-manager \
            --disablerepo "*" \
            --enablerepo "ubi-9-baseos-rpms" \
            --setopt=install_weak_deps=0 \
            --setopt=keepcache=0 \
            --best \
            --setopt=tsflags=nodocs \
        tzdata && \
    microdnf download libcurl && \
    rpm -Uvh --nodeps --replacefiles "*curl*$( uname -i ).rpm" && \
    microdnf remove -y libcurl-minimal && \
    rm -rf "*curl*$( uname -i ).rpm" && \
    groupadd \
            --system \
            --gid 1995 \
        zabbix && \
    useradd \
            --system \
            --comment "Zabbix monitoring system" \
            -g zabbix \
            -G dialout \
            --uid 1997 \
            --shell /sbin/nologin \
            --home-dir /var/lib/zabbix/ \
        zabbix && \
    chgrp zabbix /usr/bin/nmap && \
    setcap cap_net_raw+eip /usr/bin/nmap && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /usr/lib/zabbix/alertscripts && \
    mkdir -p /var/lib/zabbix/dbscripts && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /var/lib/zabbix/enc_internal && \
    mkdir -p /var/lib/zabbix/export && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    mkdir -p /usr/share/doc/zabbix-server-postgresql && \
    chown --quiet -R zabbix:root /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
    microdnf -y clean all

EXPOSE 10051/TCP

WORKDIR /var/lib/zabbix

VOLUME ["/var/lib/zabbix/snmptraps", "/var/lib/zabbix/export"]

COPY ["docker-entrypoint.sh", "/usr/bin/"]

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]

USER 1997

CMD ["/usr/sbin/zabbix_server", "--foreground", "-c", "/etc/zabbix/zabbix_server.conf"]
