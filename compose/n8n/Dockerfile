#n8n@2.7.5
FROM node:22-alpine

USER root

RUN apk add --no-cache \
    chromium \
    nss \
    freetype \
    harfbuzz \
    ca-certificates \
    ttf-freefont \
    python3 \
    py3-pip

ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true \
    PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser \
    CHROME_BIN=/usr/bin/chromium-browser \
    CHROME_PATH=/usr/lib/chromium/ \
    OCI_CLI_AUTH=instance_principal

RUN npm install -g n8n puppeteer --omit=dev --unsafe-perm=true && \
    npm cache clean --force && \
    rm -rf /root/.npm /tmp/*

RUN apk add --no-cache --virtual .build-deps \
    build-base \
    python3-dev \
    libffi-dev && \
    python3 -m venv /opt/oci && \
    /opt/oci/bin/pip install --no-cache-dir --upgrade pip && \
    /opt/oci/bin/pip install --no-cache-dir oci==2.162.0 oci-cli==3.69.0 && \
    ln -s /opt/oci/bin/oci /usr/local/bin/oci && \
    apk del .build-deps && \
    rm -rf /root/.cache /tmp/*

RUN addgroup -g 1000 node 2>/dev/null || true && \
    adduser -D -u 1000 -G node node 2>/dev/null || true && \
    mkdir -p /home/node/.n8n /home/node/scripts && \
    chown -R node:node /home/node

WORKDIR /home/node
USER node

EXPOSE 5678

CMD ["n8n"]
