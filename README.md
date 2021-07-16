# Meteor Docker image

Meteor does not provide an official Dockerfile, so this repo will try to follow all versions and release them as docker
images, so we can speed up our build Meteor Docker images where we need them and allows for easy multi-stage Dockerfile
deployment.

## Versions

Image                         | Meteor   
----------------------------- | -------
megawebmaster/meteor:1.10.1-1 | 1.10.1
megawebmaster/meteor:1.10.2-1 | 1.10.2
megawebmaster/meteor:1.11-1   | 1.11
megawebmaster/meteor:1.11.1-1 | 1.11.1
megawebmaster/meteor:1.12.1-1 | 1.12.1
megawebmaster/meteor:2.0-1    | 2.0
megawebmaster/meteor:2.1-1    | 2.1
megawebmaster/meteor:2.3-1    | 2.3
megawebmaster/meteor:2.3.2-1  | 2.3.2

# How can I use the ``megawebmaster/meteor`` image?

There are multiple ways how you can use this meteor docker image but basically you
have to add a file named ``Dockerfile`` to the root of your app and run ``docker build .``
Here are some examples of Dockerfiles that you could use:

### Simple Image for development

```dockerfile
FROM megawebmaster/meteor:1.11-1 as bundler
RUN adduser -D -u 501 -h /home/meteor meteor
ADD . /source
WORKDIR /source
USER meteor
RUN meteor npm install
CMD meteor --no-release-check 
```

### Multi-stage-building of Alpine Production Image of your Meteor app

```dockerfile
FROM megawebmaster/meteor:1.11-1 as bundler

COPY package.json /source
COPY package-lock.json /source
RUN meteor npm install

COPY . /source
WORKDIR /source
RUN meteor build --server-only --allow-superuser --directory /bundle

FROM node:12-alpine as rebuilder

RUN apk add --no-cache make gcc g++ python sudo
RUN adduser -D -u 1001 -h /home/meteor meteor

COPY --from=bundler /bundle /rebuild
WORKDIR /rebuild/bundle/programs/server
RUN npm install && npm run install --production

FROM node:12-alpine as runtime

RUN apk add --no-cache --update --quiet dumb-init
RUN adduser -D -u 1001 -h /home/meteor meteor
COPY --from=rebuilder /rebuild/bundle /webapp
WORKDIR /webapp

ENV MONGO_URL mongodb://mongo:27017/retrotool
ENV ROOT_URL http://localhost
ENV PORT 3000
ENV NODE_ENV production

EXPOSE 3000
USER meteor

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["node", "./main.js"]
```
