# Basic environment
FROM node:14

# Variables
ENV APP_PORT=3000 APP_DIR=/app
ENV DB_USER=root DB_PASSWORD=changeit

WORKDIR ${APP_DIR}

ADD main.js .
ADD package.json .
ADD package-lock.json .

ADD config config
ADD public public
ADD views views

RUN npm install

# Health
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
        CMD curl -s -f http://localhost:${APP_PORT}/health || exit 1

# Expose application port
EXPOSE ${APP_PORT}

# Run application
ENTRYPOINT [ "node", "main.js" ]