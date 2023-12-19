# https://hub.docker.com/r/nghiahl/haproxy
FROM ubuntu:focal

# Variables
ARG HAPROXY_URL=https://www.haproxy.org/download/2.8/src/haproxy-2.8.5.tar.gz
ARG HAPROXY_SHA256=3f5459c5a58e0b343a32eaef7ed5bed9d3fc29d8aa9e14b36c92c969fc2a60d9

# Install extensions
RUN set -eux; \
	groupadd --gid 99 --system haproxy; \
	useradd \
		--gid haproxy \
		--system \
		--uid 99 \
		haproxy \
	; \
	mkdir /etc/haproxy; \
	chown -R haproxy:haproxy /etc/haproxy

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
 	DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
	ca-certificates \
	gcc \
	libc6-dev \
	liblua5.3-dev \
	libpcre2-dev \
	libssl-dev \
	make \
	curl \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	curl -o /opt/haproxy.tar.gz -SL "$HAPROXY_URL"; \
	echo "$HAPROXY_SHA256 /opt/haproxy.tar.gz" | sha256sum -c; \
	mkdir -p /usr/src/haproxy; \
	tar -xzf /opt/haproxy.tar.gz -C /usr/src/haproxy --strip-components=1; \
	rm /opt/haproxy.tar.gz; \
	\
	makeOpts=' \
		TARGET=linux-glibc \
		USE_GETADDRINFO=1 \
		USE_LUA=1 LUA_INC=/usr/include/lua5.3 \
		USE_OPENSSL=1 \
		USE_PCRE2=1 USE_PCRE2_JIT=1 \
		USE_PROMEX=1 \
		\
		EXTRA_OBJS=" \
		" \
	'; \
	dpkgArch="$(dpkg --print-architecture)"; \
	case "$dpkgArch" in \
		armel) makeOpts="$makeOpts ADDLIB=-latomic" ;; \
	esac; \
	\
	nproc="$(nproc)"; \
	eval "make -C /usr/src/haproxy -j '$nproc' all $makeOpts"; \
	eval "make -C /usr/src/haproxy install-bin $makeOpts"; \
	\
	mkdir -p /etc/haproxy; \
	cp -R /usr/src/haproxy/examples/errorfiles /etc/haproxy/errors; \
	rm -rf /usr/src/haproxy; \
	\
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	find /usr/local -type f -executable -exec ldd '{}' ';' \
		| awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); print so }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-mark hold \
	curl \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false
 
WORKDIR /etc/haproxy

STOPSIGNAL SIGUSR1

# Copy sources
COPY docker-entrypoint.sh /usr/local/bin/

# Add permissions
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# Final touches
ENTRYPOINT ["docker-entrypoint.sh"]

USER haproxy

CMD ["haproxy", "-f", "/etc/haproxy/haproxy.cfg"]
