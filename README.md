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

Container linking does two things 

- sets the environment variables useful for connecting to the container

- updates the HOSTS file with the original, alias and hash_id

> docker run --link redis-server:redi alpine env
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

> docker run --link redis-server:redi alpine cat /etc/hosts
```
$ docker run --link redis-server:redi alpine cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.18.0.2      redi 7f9b6dbc7cf0 redis-server
172.18.0.3      bd187fca41f3
```

> docker run --link redis-server:redi alpine ping redi
```
PING redi (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.098 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.089 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.085 ms
^C
--- redi ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.085/0.090/0.098 ms
$ docker run --link redis-server:redi alpine ping -c 1 redi
PING redi (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.136 ms
```

## Docker Network of Container (alternate to linking)

> docker network create backend-network

> docker run -d --name redis-server --net backend-network redis

/etc/resolv.conf contains Embedded DNS server's ip. This DNS server is embedded in all containers which knows the ip corresponding to each container

> docker run --net backend-network alpine cat /etc/resolv.conf 

> docker run --net backend-network alpine ping -c 1 redis-server

> docker network create frontend-network

> docker network connect frontend-network redis-server

> docker network ls

> docker network inspect frontend-network

> docker network disconnect frontend-network redis-server
