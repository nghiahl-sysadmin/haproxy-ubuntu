# https://hub.docker.com/r/nghiahl/haproxy
FROM ubuntu:latest

# Variables
ARG haproxy_version=2.8

# Install extensions
RUN apt-get -y update && apt-get -y install \
    software-properties-common \
    && add-apt-repository ppa:vbernat/haproxy-${haproxy_version} -y \
    && apt-get -y install haproxy \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /opt/source \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false

WORKDIR /etc/haproxy

# Copy sources
COPY /project/docker-entrypoint.sh /usr/local/bin/

STOPSIGNAL SIGUSR1

# Final touches
ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["haproxy", "-f", "/etc/haproxy/haproxy.cfg"]
