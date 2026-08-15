ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest
FROM ${BUILD_FROM}

# nodejs runs the server; openssl generates the self-signed certificate on first
# start (the charger does not verify it, so no certificate authority is needed).
RUN apk add --no-cache nodejs npm openssl

WORKDIR /app
COPY package.json ./
RUN npm install --omit=dev && npm cache clean --force
COPY mqtt.js server.js check.js ./

CMD ["node", "/app/server.js"]
