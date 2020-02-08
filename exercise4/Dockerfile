# - you should use the 'node' official image, with the alpine 6.x branch
# This dir contains a Node.js app, you need to get it running in a container
FROM node:6-alpine

LABEL maintainer="jcasanella79@gmail.com"

# - this app listens on port 3000, but the container should launch on port 80
# so it will respond to http://localhost:80 on your computer
EXPOSE 3000

# - then it should use alpine package manager to install tini: 'apk add --update tini'
# - then it should create directory /usr/src/app for app files with 'mkdir -p /usr/src/app'
RUN apk update && \
    apk upgrade && \ 
    apk add --update tini && \
    mkdir -p /usr/src/app && \
    addgroup -S appgroup && \
    adduser -S appuser -S appuser -G appgroup

WORKDIR /usr/src/app

# - Node uses a "package manager", so it needs to copy in package.json file
COPY package.json package.json

# - then it needs to run 'npm install' to install dependencies from that file
# - to keep it clean and small, run 'npm cache clean --force' after above
RUN npm install && \
    npm cache clean --force

# - then it needs to copy in all files from current directory
COPY . .

USER appuser

# - then it needs to start container with command '/sbin/tini -- node ./bin/www'
# - in the end you should be using FROM, RUN, WORKDIR, COPY, EXPOSE, and CMD commands
CMD [ "tini", "--", "node", "./bin/www" ]