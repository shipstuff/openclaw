FROM ubuntu:24.04@sha256:cd1dba651b3080c3686ecf4e3c4220f026b521fb76978881737d24f200828b2b

# eMudhra CA compromise (2026-10-20): sanitize OS trust, including build tools.
RUN set -eu; \
    if command -v apk >/dev/null 2>&1; then \
      apk add --no-cache ca-certificates; \
    elif command -v apt-get >/dev/null 2>&1; then \
      apt-get update; apt-get install -y --no-install-recommends ca-certificates; \
      rm -rf /var/lib/apt/lists/*; \
    else echo 'Unsupported CA store: review before building' >&2; exit 1; fi; \
    sed -i -E 's@^(mozilla/(Comodo_AAA_Services_root|AAA_Certificate_Services|emSign_ECC_Root_CA_-_G3|emSign_Root_TLS_CA_-_G3|emSign_Root_CS_CA_-_G3)\.crt)$@!\1@' /etc/ca-certificates.conf; \
    update-ca-certificates
ENV SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt \
    REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt \
    CURL_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt \
    NODE_OPTIONS=--use-openssl-ca

RUN set -eux; \
  for attempt in 1 2 3; do \
    if apt-get update -o Acquire::Retries=3; then break; fi; \
    echo "apt-get update failed (attempt ${attempt})" >&2; \
    if [ "${attempt}" -eq 3 ]; then exit 1; fi; \
    sleep 3; \
  done; \
  apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    g++ \
    make \
    python3 \
    sudo \
  && rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash app \
  && echo "app ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/app

USER app
WORKDIR /home/app

ENV NPM_CONFIG_FUND=false
ENV NPM_CONFIG_AUDIT=false

COPY install-sh-common/cli-verify.sh /usr/local/install-sh-common/cli-verify.sh
COPY install-sh-nonroot/run.sh /usr/local/bin/openclaw-install-nonroot
RUN sudo chmod +x /usr/local/bin/openclaw-install-nonroot

ENTRYPOINT ["/usr/local/bin/openclaw-install-nonroot"]
