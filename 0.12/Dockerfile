FROM alpine:3.6

MAINTAINER Pedro Branco <branco@uphold.com> (@pedrobranco)

RUN adduser -S dash

ENV BERKELEYDB_VERSION=db-4.8.30.NC
ENV BERKELEYDB_PREFIX=/opt/${BERKELEYDB_VERSION}

ENV DASH_VERSION=0.12.2.2
ENV DASH_PREFIX=/opt/dash-${DASH_VERSION}
ENV DASH_DATA=/home/dash/.dashcore
ENV PATH=${DASH_PREFIX}/bin:$PATH

RUN apk --no-cache --virtual build-dependendencies add autoconf \
    automake \
    boost-dev \
    build-base \
    chrpath \
    file \
    gnupg \
    libevent-dev \
    libressl \
    libressl-dev \
    libtool \
    linux-headers \
    protobuf-dev \
    zeromq-dev \
  && mkdir -p /tmp/build \
  && mkdir -p /tmp/build/${DASH_VERSION} \
  && wget -O /tmp/build/${BERKELEYDB_VERSION}.tar.gz http://download.oracle.com/berkeley-db/${BERKELEYDB_VERSION}.tar.gz \
  && tar -xzf /tmp/build/${BERKELEYDB_VERSION}.tar.gz -C /tmp/build/ \
  && sed s/__atomic_compare_exchange/__atomic_compare_exchange_db/g -i /tmp/build/${BERKELEYDB_VERSION}/dbinc/atomic.h \
  && mkdir -p ${BERKELEYDB_PREFIX} \
  && cd /tmp/build/${BERKELEYDB_VERSION}/build_unix \
  && ../dist/configure --enable-cxx --disable-shared --with-pic --prefix=${BERKELEYDB_PREFIX} \
  && make install \
  && wget -P /tmp/build/${DASH_VERSION} https://github.com/dashpay/dash/archive/v${DASH_VERSION}.tar.gz \
  && echo "fd5f1576bc8ef70e5823f665b86a334937813e300f037a03bcd127b83773d771  /tmp/build/${DASH_VERSION}/v${DASH_VERSION}.tar.gz" | sha256sum -c \
  && cd /tmp/build/${DASH_VERSION} \
  && tar -xzf v${DASH_VERSION}.tar.gz -C /tmp/build \
  && cd /tmp/build/dash-${DASH_VERSION} \
  && ./autogen.sh \
  && ./configure LDFLAGS=-L${BERKELEYDB_PREFIX}/lib/ CPPFLAGS=-I${BERKELEYDB_PREFIX}/include/ \
    --prefix=${DASH_PREFIX} \
    --mandir=/usr/share/man \
    --disable-tests \
    --disable-bench \
    --disable-ccache \
    --with-gui=no \
    --with-utils \
    --with-libs \
    --with-daemon \
  && make install \
  && cd / \
  && strip ${DASH_PREFIX}/bin/dash-cli ${DASH_PREFIX}/bin/dashd ${DASH_PREFIX}/bin/dash-tx ${DASH_PREFIX}/lib/libdashconsensus.a ${DASH_PREFIX}/lib/libdashconsensus.so.0.0.0 \
  && rm -rf /tmp/build ${BERKELEYDB_PREFIX}/docs \
  && apk --no-cache --purge del build-dependendencies \
  && apk --no-cache add boost \
    boost-program_options \
    libevent \
    libressl \
    libzmq \
    su-exec

VOLUME ["/home/dash/.dashcore"]

COPY docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 9998 9999 18332 19998 19999

CMD ["dashd"]
