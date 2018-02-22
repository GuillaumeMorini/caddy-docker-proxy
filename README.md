# CADDY-DOCKER-PROXY

## WIP
We don't have any oficial release yet, but you can find the testing image here: https://hub.docker.com/r/lucaslorentz/caddy-docker-proxy/

## Introduction
Caddy docker proxy is a caddy plugin that generates caddy config files from Docker Swarm Services metadata, making caddy act as a docker proxy.

## Labels for service
| Label        | Example           | Description  |
| -------------|-------------| -----|
| caddy.address | service.test.com | list of addresses that should be proxied to that service |
| caddy.targetport | 80 | the port being serverd by the service |
| caddy.targetpath | /api | the path being served by the service |

It's possible to write any configuration using labels, like:
```
caddy.limits=7500
```
will generate:
```
limits 7500
```

And
```
caddy.limits.header=100KB
caddy.limits.body_0=/upload 100MB
caddy.limits.body_1=/profile 25KB
caddy.limits.body_2=/api 10KB
```
will generate:
```
limits {
	header 100KB
	body   /upload 100MB
	body   /profile 25KB
	body   /api 10KB
}
```

## Build it
You can use our caddy build wrapper **build.sh** and include extra plugins on https://github.com/lucaslorentz/caddy-docker-proxy/blob/master/main.go#L5

Or, you can build from caddy repository and import  **caddy-docker-proxy** plugin on file https://github.com/mholt/caddy/blob/master/caddy/caddymain/run.go :
```
import (
  _ "github.com/lucaslorentz/caddy-docker-proxy/plugin"
)
```

## Try it

Create caddy network:
```
docker network create --driver overlay caddy
```

Run caddy proxy:
```
docker service create --name caddy-docker-proxy --constraint=node.role==manager --publish 2015:2015 --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock --network caddy lucaslorentz/caddy-docker-proxy -log stdout
```

Create services:
```
docker service create --network caddy -l caddy.address=whoami0.caddy-proxy -l caddy.targetport=8000 -l caddy.tls=off --name whoami0 jwilder/whoami


docker service create --network caddy -l caddy.address=whoami1.caddy-proxy -l caddy.targetport=8000 -l caddy.tls=off --name whoami1 jwilder/whoami
```

Access them through the proxy:
```
curl -H Host:whoami0.caddy-proxy http://localhost:2015

curl -H Host:whoami1.caddy-proxy http://localhost:2015
```
