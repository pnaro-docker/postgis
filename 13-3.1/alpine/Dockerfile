FROM postgres:13-alpine

LABEL maintainer="PostGIS Project - https://postgis.net"

ENV POSTGIS_VERSION 3.1.4
ENV POSTGIS_SHA256 dfcbad0c6090c80bc59d3ea77d1adc4b3ade533a403761b4af6d9a44be1a6e48

#Temporary fix:
#   for PostGIS 2.* - building a special geos
#   reason:  PostGIS 2.5.5 is not working with GEOS 3.9.*
ENV POSTGIS2_GEOS_VERSION tags/3.8.2

COPY ./postgis-3.1.4.tar.gz postgis.tar.gz

RUN set -eux

RUN apk add --no-cache --virtual .fetch-deps \
        ca-certificates \
        openssl \
        tar

#RUN wget -O postgis.tar.gz "https://github.com/postgis/postgis/archive/$POSTGIS_VERSION.tar.gz"
RUN echo "$POSTGIS_SHA256 *postgis.tar.gz" | sha256sum -c -
RUN mkdir -p /usr/src/postgis
RUN tar \
        --extract \
        --file postgis.tar.gz \
        --directory /usr/src/postgis \
        --strip-components 1
RUN rm postgis.tar.gz

RUN apk add --no-cache --virtual .build-deps \
        autoconf \
        automake \
        clang-dev \
        file \
        g++ \
        gcc \
        gdal-dev \
        gettext-dev \
        json-c-dev \
        libtool \
        libxml2-dev \
        llvm11-dev \
        make \
        pcre-dev \
        perl \
        proj-dev \
        protobuf-c-dev

# GEOS setup
RUN apk add --no-cache --virtual .build-deps-geos geos-dev cunit-dev
# build PostGIS

WORKDIR /usr/src/postgis
RUN gettextize
RUN ./autogen.sh
RUN ./configure --with-pcredir="$(pcre-config --prefix)"
RUN make -j$(nproc)
RUN make install
# regress check
#RUN mkdir /tempdb
#RUN chown -R postgres:postgres /tempdb
#RUN su postgres -c 'pg_ctl -D /tempdb init'
#RUN su postgres -c 'pg_ctl -D /tempdb start' \
#    && cd /usr/src/postgis/regress \
#    && make -j$(nproc) check RUNTESTFLAGS=--extension   PGUSER=postgres \
#    && su postgres -c 'pg_ctl -D /tempdb --mode=immediate stop'
#RUN rm -rf /tempdb
#RUN rm -rf /tmp/pgis_reg
# add .postgis-rundeps
RUN apk add --no-cache --virtual .postgis-rundeps \
        gdal \
        json-c \
        libstdc++ \
        pcre \
        proj \
        protobuf-c
# Geos setup
RUN apk add --no-cache --virtual .postgis-rundeps-geos geos
# clean

WORKDIR /
RUN rm -rf /usr/src/postgis
RUN apk del .fetch-deps .build-deps .build-deps-geos

COPY ./initdb-postgis.sh /docker-entrypoint-initdb.d/10_postgis.sh
COPY ./update-postgis.sh /usr/local/bin