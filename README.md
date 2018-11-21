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

## Persisting Data using Data Volumes

> docker run -v /docker/redis-data:/data --name r1 -d redis --appendonly yes

Pipe the redis data into r1 redis instance

> echo "SET Key0 Value0" | docker exec -i r1 redis-cli --pipe

> ls /docker/redis-data
```
appendonly.aof
```

Use the directory in another container

> docker run -v /docker/redis-data:/backup ubuntu ls /backup
```
appendonly.aof
```

### Sharing volumes

Another alternative to above is as below. New container can access /data volume of r1 which is actually mapped to /docker/redis-data

> docker run --volumes-from r1 -it  ubuntu ls /data

### Readonly volumes (ro)

> docker run -v /docker/redis-data:/data:ro -it ubuntu rm -rf /data

## Logging 

To redirect logs to syslog use --log-driver syslog

To disable logging use --log-driver none

## Restart containers on crash (exit <> 0)

Default behavior : no restart

restart attempt thrice : --restart on-failure:3

restart unlimited times (always) : --restart always

## Load Balancing using NGINX Proxy (jwilder)

Based on https://github.com/jwilder/nginx-proxy

> docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro -e DEFAULT_HOST example.com --name nginx jwilder/nginx-proxy

> curl http://docker (returns 503 as there's no server configured)

> docker run -d -p 80 -e VIRTUAL_HOST example.com katacoda/docker-http-server (registers to nginx proxy)

> curl http://docker (returns the host name where http server is running)

> docker run -d -p 80 -e VIRTUAL_HOST example.com katacoda/docker-http-server (registers to nginx proxy)

subsequent calls to example.com would be proxied to the above servers in round-robin fashion

> docker exec nginx cat /etc/nginx/conf.d/default.conf (to view the automatically updated configuration of nginx proxy based on docker-gen)

## Container Oschestration using Docker Compose

docker-compose.yaml
```
web:
  build .
  links:
    - redis-server
  ports:
    - "3000"
    
 redis-server:
   image: redis:alpine
   volumes:
     - /var/redis/data:/data
```

> docker-compose up -d

> docker-compose ps

> docker-compose scale web=3 (starts two more containers to scale up)

> docker-compose scale web=1 (stops two containers to scale down)

> docker-compose stop (stops all containers)

> docker-compose rm (removes all containers)

## Container stats (CPU, Memory, Network IO)

> docker stats nginx (one particular container)

> docker ps -q | xargs docker stats (all container stats)


# Swarm Mode

Turn single host docker host into multi-host Docker Swarm Mode. By default, docker works as an isolated single-node. All containers are only deployed onto the engine. Swarm Mode turns it into a multi-host cluster-aware engine.

### Managers and Workers 
Managers schedule which containers to run where. Workers execute a bunch of tasks. Managers are also workers. 

> docker swarm --help

> docker swarm init

- Node initializing the swarm mode becomes the manager of the cluster.

- Note down the token for adding nodes to the swarm

From another node, join the cluster as below

> token=$(docker -H 172.17.0.2:2345 swarm join-token -q worker) && echo $token

> docker swarm join 172.17.0.2:2377 --token $token

From the manager node, enter below to view the status of the cluster

> docker node ls

### Overlay Network

> docker network create -d overlay skynet

Containers registered to the above skynet network will be able to communicate to each other regardless of which host they belong to. [VXLAN - Virtual Extensible LAN]

### Create a service with two replicas of http server load-balanced and spread across evenly on the network 

It internally creates and manages containers on nodes in the network

> docker service create --name http --network skynet --replicas 2 -p 80:80 katacoda/docker-http-server

> docker service ls

list containers on each host using 

> docker ps

> curl docker (returns the host which served the request, run multiple times to see the load-balancing working in round-robin fashion)

Key features of a service
- Load balanced app
- create required number of replicas
- Manage containers in the network
- Scaling

> docker service ps http (inspect the state and health of cluster and running apps)

### Scale the service

> docker service --help

> docker service scale http=5

> docker service ps http (you will see 3 containers running on one host and 2 containers on the other)



