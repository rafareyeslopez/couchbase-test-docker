FROM ubuntu:14.04

MAINTAINER Couchbase Docker Team <docker@couchbase.com>

# Install dependencies:
#  runit: for container process management
#  wget: for downloading .deb
#  python-httplib2: used by CLI tools
#  chrpath: for fixing curl, below
# Additional dependencies for system commands used by cbcollect_info:
#  lsof: lsof
#  lshw: lshw
#  sysstat: iostat, sar, mpstat
#  net-tools: ifconfig, arp, netstat
#  numactl: numactl
RUN apt-get update && \
    apt-get install -yq runit wget python-httplib2 chrpath \
    lsof lshw sysstat net-tools numactl  && \
    apt-get autoremove && apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV CB_VERSION=4.5.0 \
    CB_RELEASE_URL=http://packages.couchbase.com/releases \
    CB_PACKAGE=couchbase-server-enterprise_4.5.0-ubuntu14.04_amd64.deb \
    CB_SHA256=441398302210c0d73f27bdab741b471fc9da116bf45f521b314345f04560716e \
    PATH=$PATH:/opt/couchbase/bin:/opt/couchbase/bin/tools:/opt/couchbase/bin/install

# Create Couchbase user with UID 1000 (necessary to match default
# boot2docker UID)
RUN groupadd -g 1000 couchbase && useradd couchbase -u 1000 -g couchbase -M

# Install couchbase
RUN wget -N $CB_RELEASE_URL/$CB_VERSION/$CB_PACKAGE && \
    echo "$CB_SHA256  $CB_PACKAGE" | sha256sum -c - && \
    dpkg -i ./$CB_PACKAGE && rm -f ./$CB_PACKAGE  && \
    sleep 10 && /opt/couchbase/bin/couchbase-cli  cluster-init -c $HOSTNAME:8091 --cluster-username=Administrator  --cluster-password=password --cluster-port=8091 --cluster-ramsize=384 --cluster-index-ramsize=384 --services=data,index,query && /opt/couchbase/bin/couchbase-cli bucket-create -c $HOSTNAME:8091 --bucket=default --bucket-type=couchbase --bucket-port=11211  --bucket-ramsize=100  --bucket-replica=0 -u Administrator -p password && /opt/couchbase/bin/cbdocloader -n $HOSTNAME:8091 -u Administrator -p password -b beer-sample /opt/couchbase/samples/beer-sample.zip


# Add runit script for couchbase-server
COPY scripts/run /etc/service/couchbase-server/run

# Add dummy script for commands invoked by cbcollect_info that
# make no sense in a Docker container
COPY scripts/dummy.sh /usr/local/bin/
RUN ln -s dummy.sh /usr/local/bin/iptables-save && \
    ln -s dummy.sh /usr/local/bin/lvdisplay && \
    ln -s dummy.sh /usr/local/bin/vgdisplay && \
    ln -s dummy.sh /usr/local/bin/pvdisplay

# Fix curl RPATH
RUN chrpath -r '$ORIGIN/../lib' /opt/couchbase/bin/curl

# Add bootstrap script
COPY scripts/entrypoint.sh /
RUN chmod +x /entrypoint.sh
RUN chmod +x /usr/local/bin/dummy.sh
RUN chmod +x /etc/service/couchbase-server/run
ENTRYPOINT ["/entrypoint.sh"]
CMD ["couchbase-server"]

# 8091: Couchbase Web console, REST/HTTP interface
# 8092: Views, queries, XDCR
# 8093: Query services (4.0+)
# 8094: Full-text Serarch (4.5+)
# 11207: Smart client library data node access (SSL)
# 11210: Smart client library/moxi data node access
# 11211: Legacy non-smart client library data node access
# 18091: Couchbase Web console, REST/HTTP interface (SSL)
# 18092: Views, query, XDCR (SSL)
# 18093: Query services (SSL) (4.0+)
EXPOSE 8091 8092 8093 8094 11207 11210 11211 18091 18092 18093
