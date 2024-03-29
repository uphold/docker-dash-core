FROM debian:stable-slim

LABEL maintainer.0="João Fonseca (@joaopaulofonseca)" \
  maintainer.1="Pedro Branco (@pedrobranco)" \
  maintainer.2="Rui Marinho (@ruimarinho)"

RUN useradd -r dash \
  && apt-get update -y \
  && apt-get install -y curl gnupg unzip \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && set -ex \
  && for key in \
    B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    29590362EC878A81FD3C202B52527BEDABE87984 \
  ; do \
    gpg --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
    gpg --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --keyserver keys.openpgp.org --recv-keys "$key" ; \
  done

ENV GOSU_VERSION=1.10

RUN curl -o /usr/local/bin/gosu -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture) \
  && curl -o /usr/local/bin/gosu.asc -fSL https://github.com/tianon/gosu/releases/download/${GOSU_VERSION}/gosu-$(dpkg --print-architecture).asc \
  && gpg --verify /usr/local/bin/gosu.asc \
  && rm /usr/local/bin/gosu.asc \
  && chmod +x /usr/local/bin/gosu

ARG TARGETPLATFORM
ENV DASH_VERSION=20.0.4
ENV DASH_FOLDER_VERSION=20.0.4
ENV DASH_DATA=/home/dash/.dashcore \
  PATH=/opt/dashcore-${DASH_FOLDER_VERSION}/bin:$PATH
RUN set -ex \
  && if [ "${TARGETPLATFORM}" = "linux/amd64" ]; then export TARGETPLATFORM=x86_64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ]; then export TARGETPLATFORM=aarch64-linux-gnu; fi \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ]; then export TARGETPLATFORM=arm-linux-gnueabihf; fi \
  && curl -SLO https://github.com/dashpay/dash/releases/download/v${DASH_VERSION}/SHA256SUMS.asc \
  && curl -SLO https://github.com/dashpay/dash/releases/download/v${DASH_VERSION}/dashcore-${DASH_VERSION}-${TARGETPLATFORM}.tar.gz \
  && curl -SLO https://github.com/dashpay/dash/releases/download/v${DASH_VERSION}/dashcore-${DASH_VERSION}-${TARGETPLATFORM}.tar.gz.asc \
  && gpg --verify dashcore-${DASH_VERSION}-${TARGETPLATFORM}.tar.gz.asc \
  && tar -xzf dashcore-${DASH_VERSION}-${TARGETPLATFORM}.tar.gz -C /opt \
  && rm *.tar.gz

VOLUME ["/home/dash/.dashcore"]

COPY docker-entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

EXPOSE 9998 9999 19898 19998 19999

CMD ["dashd"]
