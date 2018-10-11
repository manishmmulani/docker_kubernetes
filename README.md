# docker

> docker --help

> docker search redis

> docker run -d redis
a9d4ae7faba24774c5b72905c6d8d3a0e09277d6440e129115769bc75299427e

by default above runs the latest version
to run a particular version use

> docker run -d redis:3.2

> docker images
lists images

> docker ps 
lists containers

> docker inspect a9d4ae7faba24774c5b72905c6d8d3a0e09277d6440e129115769bc75299427e | less

> docker logs a9d4ae7faba24774c5b72905c6d8d3a0e09277d6440e129115769bc75299427e

To run named container and expose port of the app to be able to access -p <host-port>:<container-port>
> docker run -d --name redisHostPort -p 127.0.0.1:6379:6379 redis:latest

To run named container and expose app on a dynamic port (not on fixed port)
> docker run -d --name redisDynamic -p 6379 redis:latest

To know the port, either run 'docker ps' or 'docker port redisDynamic'


# Docker Images

alpine version of nginx is the BASE image for the image  we are trying to build

## DockerFile nginx
```
FROM nginx:alpine
COPY . /usr/share/nginx/html
```

> docker build -t webserver-image:v1 .


- Docker caches the result of each line to make subsequent builds run faster. To invalidate the cache of a certain line / below lines, add a command (such as copy package.json below) so that 'docker build' runs them when something changes there.

- WORKDIR ensures future commands run relative to the work directory (eg: RUN npm install and CMD ["npm", "start"] below will be run from /src/app/)

- Difference b/w RUN and CMD : RUN is executed during building the image. CMD can be overriden using docker run and CMD will be executed within the container to launch the app in general.

## DockerFile nodejs app
```
FROM node:10-alpine
RUN mkdir -p /src/app
WORKDIR "/src/app"

COPY package.json /src/app/package.json 
RUN npm install
COPY . /src/app/
EXPOSE 3000

CMD ["npm", "start"]
```
