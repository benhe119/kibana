FROM alpine:3.8

LABEL maintainer "https://github.com/blacktop"

ENV VERSION 6.5.4
ENV DOWNLOAD_URL https://artifacts.elastic.co/downloads/kibana
ENV TARBAL "${DOWNLOAD_URL}/kibana-oss-${VERSION}-linux-x86_64.tar.gz"
ENV TARBALL_ASC "${DOWNLOAD_URL}/kibana-oss-${VERSION}-linux-x86_64.tar.gz.asc"
ENV TARBALL_SHA "65c4ab427ae2a2486b71520893eeb78d8628bf3e6fd47c0d576ca4b6bfa8f9058ecf957ef488fd47260c8295ea49952e1b411d6a6c199f26d74e6a84cfcf7309"
ENV GPG_KEY "46095ACC8548582C1A2699A9D27D666CD88E42B4"

ENV PATH /usr/share/kibana/bin:$PATH

RUN apk add --no-cache bash nodejs su-exec
RUN apk add --no-cache -t .build-deps wget ca-certificates gnupg openssl fontconfig freetype \
  && set -ex \
  && cd /tmp \
  && echo "===> Install Kibana..." \
  && wget --progress=bar:force -O kibana.tar.gz "$TARBAL"; \
  if [ "$TARBALL_SHA" ]; then \
  echo "$TARBALL_SHA *kibana.tar.gz" | sha512sum -c -; \
  fi; \
  if [ "$TARBALL_ASC" ]; then \
  wget --progress=bar:force -O kibana.tar.gz.asc "$TARBALL_ASC"; \
  export GNUPGHOME="$(mktemp -d)"; \
  ( gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
  || gpg --keyserver pgp.mit.edu --recv-keys "$GPG_KEY" \
  || gpg --keyserver keyserver.pgp.com --recv-keys "$GPG_KEY" ); \
  gpg --batch --verify kibana.tar.gz.asc kibana.tar.gz; \
  rm -rf "$GNUPGHOME" kibana.tar.gz.asc || true; \
  fi; \
  tar -xf kibana.tar.gz \
  && ls -lah \
  && mv kibana-$VERSION-linux-x86_64 /usr/share/kibana \
  && adduser -DH -s /sbin/nologin kibana \
  # usr alpine nodejs and not bundled version
  && bundled='NODE="${DIR}/node/bin/node"' \
  && apline_node='NODE="/usr/bin/node"' \
  && sed -i "s|$bundled|$apline_node|g" /usr/share/kibana/bin/kibana-plugin \
  && sed -i "s|$bundled|$apline_node|g" /usr/share/kibana/bin/kibana \
  && rm -rf /usr/share/kibana/node \
  && chown -R kibana:kibana /usr/share/kibana \
  && rm -rf /tmp/* \
  && apk del --purge .build-deps

RUN echo "===> Install Kibana Plugin..." \
  && kibana-plugin install \
  https://github.com/maliceio/malice-kibana-plugin/releases/download/v$VERSION/malice-$VERSION.zip

COPY --chown=kibana:kibana config/kibana.yml /usr/share/kibana/config/kibana.yml
COPY entrypoint.sh /

WORKDIR /usr/share/kibana

EXPOSE 5601

ENTRYPOINT ["/entrypoint.sh"]
CMD ["kibana"]
