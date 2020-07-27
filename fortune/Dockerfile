# Build the environment
FROM node:14

# Set the environmental variables
ENV APP_DIR=/app APP_PORT=3000

# Creates (if necessary) and sets the working directory
WORKDIR ${APP_DIR}

# Copy content of fortune to /app
ADD main.js .
ADD package.json .
ADD package-lock.json .

# Copy the directories
ADD public public
ADD views views

# install dependencies
RUN npm install

# Add a way to check container health through docker ps
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
        CMD curl -s -f http://localhost:${APP_PORT}/health || exit 1

# Expose application port
EXPOSE ${APP_PORT}

# Run application
ENTRYPOINT [ "node", "main.js" ]