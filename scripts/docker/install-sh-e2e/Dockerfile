FROM node:22.23.4-bookworm-slim@sha256:de170b39439eb0e5a3444ad7d60b015614a381e10fecae2e30ec9e735122f38d

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

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    bash \
    ca-certificates \
    curl \
    git \
  && rm -rf /var/lib/apt/lists/*

COPY run.sh /usr/local/bin/openclaw-install-e2e
RUN chmod +x /usr/local/bin/openclaw-install-e2e

RUN useradd --create-home --shell /bin/bash appuser
USER appuser

ENTRYPOINT ["/usr/local/bin/openclaw-install-e2e"]
