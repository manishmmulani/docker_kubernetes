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

## Data Containers

-v for where other containers read / write data
busybox - base container 

> docker create -v /config --name dataContainer busybox

Copy config.conf from client directory to dataContainer

> docker cp config.conf dataContainer:/config/

Now launch a new container (ubuntu image) and mount the volume from dataContainer

> docker run --volumes-from dataContainer ubuntu ls /config

config.conf

Exporting dataContainer into a tar file to be able to move to a  different machine

> docker export dataContainer > dataContainer.tar

Importing tar

> docker import dataContainer.tar

## Container linking

eg: Your app needs to connect to data store which is running on another container

> docker run -d --name redis-server redis

> docker run --link redis-server:alias alpine env
```
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=9c470caa0608
REDI_PORT=tcp://172.18.0.2:6379
REDI_PORT_6379_TCP=tcp://172.18.0.2:6379
REDI_PORT_6379_TCP_ADDR=172.18.0.2
REDI_PORT_6379_TCP_PORT=6379
REDI_PORT_6379_TCP_PROTO=tcp
REDI_NAME=/alpine-app/redi
REDI_ENV_GOSU_VERSION=1.10
REDI_ENV_REDIS_VERSION=4.0.11
REDI_ENV_REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-4.0.11.tar.gz
REDI_ENV_REDIS_DOWNLOAD_SHA=fc53e73ae7586bcdacb4b63875d1ff04f68c5474c1ddeda78f00e5ae2eed1bbb
HOME=/root
```
