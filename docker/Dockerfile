FROM debian:bookworm-slim

ARG UID=101
ARG GID=101
ARG VERSION
ARG KEYS

LABEL org.opencontainers.image.source="https://github.com/kub0-ai/dogecoin-core"
LABEL org.opencontainers.image.description="Dogecoin Core v${VERSION}"
LABEL org.opencontainers.image.licenses="MIT"

RUN groupadd --gid ${GID} dogecoin \
  && useradd --create-home --no-log-init -u ${UID} -g ${GID} dogecoin \
  && apt-get update -y \
  && apt-get install -y curl gnupg gosu ca-certificates \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ARG TARGETPLATFORM
ENV DOGECOIN_VERSION=${VERSION}
ENV DOGECOIN_DATA=/home/dogecoin/.dogecoin
ENV PATH=/opt/dogecoin-${DOGECOIN_VERSION}/bin:$PATH

RUN set -ex \
  && case "${TARGETPLATFORM}" in \
       linux/amd64) ARCH=x86_64-linux-gnu ;; \
       linux/arm64) ARCH=aarch64-linux-gnu ;; \
       *) echo "Unsupported platform: ${TARGETPLATFORM}" && exit 1 ;; \
     esac \
  && curl -SLO https://github.com/dogecoin/dogecoin/releases/download/v${DOGECOIN_VERSION}/SHA256SUMS.asc \
  && if [ -n "${KEYS}" ]; then \
       echo "Importing provided keys..."; \
       for key in ${KEYS}; do \
         gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
         gpg --batch --keyserver keys.openpgp.org --recv-keys "$key" || \
         gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" ; \
       done; \
     else \
       echo "Auto-discovering signing keys from SHA256SUMS.asc..."; \
       AUTO_KEYS=$(gpg --list-packets SHA256SUMS.asc 2>/dev/null | grep "keyid" | awk '{print $NF}' | sort -u); \
       for key in $AUTO_KEYS; do \
         gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
         gpg --batch --keyserver keys.openpgp.org --recv-keys "$key" || \
         gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" ; \
       done; \
     fi \
  && if ! gpg --verify --status-fd=1 SHA256SUMS.asc 2>/dev/null | grep -qE "GOODSIG|EXPKEYSIG"; then \
       echo "ERROR: No valid GPG signature found in SHA256SUMS.asc"; exit 1; \
     fi \
  && curl -SLO https://github.com/dogecoin/dogecoin/releases/download/v${DOGECOIN_VERSION}/dogecoin-${DOGECOIN_VERSION}-${ARCH}.tar.gz \
  && grep " dogecoin-${DOGECOIN_VERSION}-${ARCH}.tar.gz" SHA256SUMS.asc | sha256sum -c - \
  && tar -xzf *.tar.gz -C /opt \
  && rm *.tar.gz *.asc

COPY docker-entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

VOLUME ["/home/dogecoin/.dogecoin"]
EXPOSE 22555 22556

ENTRYPOINT ["/entrypoint.sh"]
RUN dogecoind -version | grep "Dogecoin Core Daemon version v${DOGECOIN_VERSION}"
CMD ["dogecoind", "-printtoconsole"]
