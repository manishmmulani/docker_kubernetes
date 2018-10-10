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

