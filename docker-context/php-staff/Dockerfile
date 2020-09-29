FROM alpine:3.10
LABEL maintainer="php-staff-dev@gmo.jp"

RUN adduser -D -H -s /bin/sh -u 1000 nhandd

ENV EXTENSION_DIR=/usr/lib/php7/modules/
ENV TIMEZONE Asia/Tokyo
ENV ORACLE_HOME=/usr/local/lib/instantclient_11_2
ENV NLS_LANG=Japanese_Japan.JA16EUC
ENV LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH
ENV PATH=$ORACLE_HOME:$PATH
ENV NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS'

# limit
COPY ./templates/limits/limits.conf /etc/security/
COPY ./templates/limits/90-nproc.conf /etc/security/limits.d/

## pdo_oci
ADD ./download/instantclient-basic-linux.x64-11.2.0.4.0.zip /usr/local/src/
ADD ./download/instantclient-sdk-linux.x64-11.2.0.4.0.zip /usr/local/src/
RUN unzip /usr/local/src/instantclient-basic-linux.x64-11.2.0.4.0.zip -d /usr/local/lib \
    && unzip /usr/local/src/instantclient-sdk-linux.x64-11.2.0.4.0.zip -d /usr/local/lib \
    && ln -s /usr/local/lib/instantclient_11_2/libclntsh.so.11.1 /usr/local/lib/instantclient_11_2/libclntsh.so \
    && rm /usr/local/src/instantclient-basic-linux.x64-11.2.0.4.0.zip \
    && rm /usr/local/src/instantclient-sdk-linux.x64-11.2.0.4.0.zip
## oci8 and pdo_oci
COPY ./templates/pdo_oci/oracle.sh /etc/profile.d/

## php
RUN apk --update add \
    abuild \
    autoconf \
    automake \
    binutils \
    build-base \
    ca-certificates \
    curl-dev \
    file \
    gcc \
    libc-dev \
    gettext-dev \
    libaio-dev \
    libcurl \
    libtirpc-dev \
    libtool \
    libxml2-dev \
    libedit-dev \
    libmcrypt-dev \
    libmemcached-dev\
    libzip-dev \
    cyrus-sasl-dev \
    openrc \
    re2c \
    tzdata \
    procps \
    --no-cache \
    && cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && chmod 0644 /etc/localtime \
    && wget https://github.com/thkukuk/libnsl/archive/v1.2.0/libnsl-1.2.0.tar.gz \
    && tar xf libnsl-1.2.0.tar.gz \
    && cd libnsl-1.2.0 \
    && sh autogen.sh \
    && ./configure -C \
    && make > /dev/null \
    && make install > /dev/null \
    && ln -s /usr/lib/libnsl.so.2 /lib/libnsl.so.1 \
    && cd / \
    && rm libnsl-1.2.0.tar.gz \
    && rm -rf libnsl-1.2.0 \
    && mkdir -p /usr/local/src \
    && wget http://mirror.cogentco.com/pub/php/php-7.3.18.tar.bz2 -O /usr/local/src/php-7.3.18.tar.bz2 \
    && tar jxf /usr/local/src/php-7.3.18.tar.bz2 -C /usr/local/src/ --no-same-owner 

COPY ./download/oci_statement.c /usr/local/src/php-7.3.18/ext/pdo_oci

RUN cd /usr/local/src/php-7.3.18/ \
    && ./configure \
--build=x86_64-alpine-linux-musl \
--host=x86_64-alpine-linux-musl \
--prefix=/usr \
--libdir=/usr/lib/php7 \
--datadir=/usr/share/php7 \
--sysconfdir=/etc/php7 \
--localstatedir=/var \
--with-layout=GNU \
--with-pic \
--with-pear=/usr/share/php7 \
--with-config-file-path=/etc/php7 \
--with-config-file-scan-dir=/etc/php7/conf.d \
--disable-short-tags \
--enable-ctype \
--with-curl \
--enable-fpm \
--with-iconv \
--enable-json \
--enable-mbstring \
--with-mcrypt \
--with-mysqli=mysqlnd \
--enable-xml \
--enable-libxml \
--with-libxml-dir=/usr \
--enable-pdo \
--with-pdo-mysql=mysqlnd \
--enable-simplexml \
--enable-opcache \
--enable-tokenizer \
--enable-xmlreader \
--enable-ftp \
--enable-phar \
--enable-posix \
--enable-fileinfo \
--enable-zip \
--with-zlib \
--with-zlib-dir=/usr \
--disable-phpdbg \
--enable-embed \
--with-litespeed \
--without-pdo-sqlite \
--with-libedit \
--with-openssl \
--with-litespeed \
build_alias=x86_64-alpine-linux-musl \
host_alias=x86_64-alpine-linux-musl \
    && make -j5 > /dev/null \
    && make install \
    && cp php.ini-production /etc/php7/php.ini \
    && chmod 644 /etc/php7/php.ini \
    && mkdir -p /etc/php7/conf.d/ \
    && mkdir -p /etc/php.d/ \
    && mkdir -p /etc/php7/php-fpm.d \
    && mv /etc/php7/php-fpm.conf.default /etc/php7/php-fpm.conf \
    && echo "env[NLS_LANG]=Japanese_Japan.JA16EUC" >> /etc/php7/php-fpm.conf \
    && echo "env[NLS_DATE_FORMAT]='YYYY-MM-DD HH24:MI:SS'" >> /etc/php7/php-fpm.conf \
    && sed -i -e"s/expose = On/expose = Off/g" /etc/php7/php.ini \
    && echo 'extension=oci8.so' >> /etc/php7/conf.d/oci8.ini \
    && echo 'extension=pdo_oci.so' >> /etc/php7/conf.d/pdo_oci.ini \
    && cd /usr/local/src/php-7.3.18/ext/oci8/ \
    && phpize \
    && ./configure --with-oci8=shared,instantclient,/usr/local/lib/instantclient_11_2,11.2 \
    && make > /dev/null \
    && make install \
    && cd /usr/local/src/php-7.3.18/ext/pdo_oci \
    && phpize \
    && ./configure --with-pdo_oci=shared,instantclient,/usr/local/lib/instantclient_11_2,11.2 \
    && make > /dev/null \
    && make install \
    && rm /usr/local/src/php-7.3.18.tar.bz2 \
    && sed -i "s|;date.timezone =.*|date.timezone = ${TIMEZONE}|" /etc/php7/php.ini \
    && rm -rf /usr/local/src/php-7.3.18 \
    && pear list-upgrades \
    && pecl install -n solr \
    && echo 'extension=solr.so' >> /etc/php7/conf.d/solr.ini \
    && pecl install -n memcached \
    && echo 'extension=memcached.so' >> /etc/php7/conf.d/memcached.ini \
    && apk del \
abuild \
automake \
autoconf \
build-base \
file \
libtool \
openrc \
re2c \
gcc \
tzdata 

CMD php-fpm -F;