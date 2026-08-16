ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest
FROM ${BUILD_FROM}

# nodejs runs the server; openssl generates the self-signed certificate on first
# start (the charger does not verify it, so no certificate authority is needed).
RUN apk add --no-cache nodejs npm openssl

WORKDIR /app
# The lock file comes along so the image installs the same dependency tree CI
# validated with `npm ci`, rather than whatever is newest at build time.
COPY package.json package-lock.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY mqtt.js server.js check.js ./

CMD ["node", "/app/server.js"]
