FROM ghcr.io/openfaas/of-watchdog:0.10.7 AS watchdog
FROM node:22 AS ship

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

# Turn down the verbosity to default level.
ENV NPM_CONFIG_LOGLEVEL warn

RUN chmod 777 /tmp

RUN mkdir -p /home/node/function

# Entrypoint
WORKDIR /home/node
COPY --chown=node:node package.json ./

# This ordering means the npm installation is cached for the outer function handler.
RUN npm i

# Copy outer function handler
COPY --chown=node:node index.js ./

# COPY function node packages and install, adding this as a separate
# entry allows caching of npm install

WORKDIR /home/node/function
COPY --chown=node:node function/*.json ./

RUN npm install
RUN npx -y playwright install-deps && \
    npx -y playwright install chromium-headless-shell

# COPY function files and folders
COPY --chown=node:node function/ ./

# Set correct permissions to use non root user
WORKDIR /home/node/

ENV cgi_headers="true"
ENV fprocess="node --inspect=0.0.0.0:9229 index.js"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"

ENV exec_timeout="10s"
ENV write_timeout="15s"
ENV read_timeout="15s"

ENV prefix_logs="false"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1
EXPOSE 9229

CMD ["fwatchdog"]
