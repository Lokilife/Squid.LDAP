FROM alpine:3.20 AS builder

RUN apk add --no-cache \
    alpine-sdk \
    openldap-dev \
    linux-headers \
    libtool \
    automake \
    autoconf \
    wget

ARG SQUID_VERSION=7.6
RUN wget -O /tmp/squid-${SQUID_VERSION}.tar.gz \
    https://github.com/squid-cache/squid/archive/refs/tags/SQUID_${SQUID_VERSION//./_}.tar.gz && \
    cd /tmp && \
    tar -xzf squid-${SQUID_VERSION}.tar.gz && \
    cd squid-SQUID_${SQUID_VERSION//./_} && \
    autoreconf -fvi && \
    ./configure --prefix=/usr \
        --enable-auth-basic="LDAP" \
        --with-openldap && \
    cd lib && \
    make && \
    cd ../compat && \
    make && \
    cd ../src/auth/basic/LDAP && \
    make && \
    cp basic_ldap_auth /tmp/

FROM alpine:3.20 AS final

RUN apk add --no-cache \
    squid \
    su-exec \
    openldap-clients

COPY --from=builder /tmp/basic_ldap_auth /usr/lib/squid/

RUN mkdir -p /var/cache/squid /var/log/squid && \
    chown squid:squid /var/cache/squid /var/log/squid

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 3128

ENTRYPOINT ["/entrypoint.sh"]
