# NGINX feature test on Docker

[![Build Status](https://travis-ci.org/2stacks/docker-nginx-lb.svg?branch=master)](https://travis-ci.org/2stacks/docker-nginx-lb)
[![GitHub stars](https://img.shields.io/github/stars/badges/shields.svg?style=social&label=Star)](https://github.com/2stacks/docker-nginx-lb)
[![Github Releases](https://img.shields.io/github/downloads/atom/atom/latest/total.svg)](https://github.com/2stacks/docker-nginx-lb)
[![ImageLayers](https://images.microbadger.com/badges/image/2stacks/nginx-lb.svg)](https://microbadger.com/images/2stacks/nginx-lb)

Builds an NGINX Loadbalancer to balance the load of two NGINX webservers. Webserver content can be updated on the fly by updating files insside of 'my-site' directory. This project was composed in a folder named 'dock-cp-nginx-lb' which resulted in a project-name of dockcpnginxlb.  If your folder name differs use your project-name prefix when running validation commands.  

This documentation doesn't cover instalation of docker or docker compose view the following links for more information.  To get started clone this GitHub repository to a host with docker and docker-compose.

* [Install Docker](https://docs.docker.com/engine/installaion)
* [Install Docker Compose](https://docs.docker.com/compose/overview)

#### Repo Links

* Docker Registry @ [2stacks/nginx-lb](https://hub.docker.com/r/2stacks/nginx-lb)
* GitHub @ [2stacks/docker-nginx-lb](https://github.com/2stacks/docker-nginx-lb)

## Quick instructions:

- Run docker-compose to build the environment.

```bash
docker-compose up --build -d
```
	Creating network "dockcpnginxlb_default" with the default driver
	Building nginx-lb
	Step 1/3 : FROM nginx
	 ---> 5766334bdaa0
	Step 2/3 : LABEL maintainer "2stacks@2stacks.net"
	 ---> Using cache
	 ---> 924cf9fc4f7b
	Step 3/3 : COPY . /etc/nginx/
	 ---> Using cache
	 ---> 627012489849
	Successfully built 627012489849
	Creating dockcpnginxlb_nginx-02_1
	Creating dockcpnginxlb_nginx-01_1
	Creating dockcpnginxlb_nginx-lb_1

## Validation:

- Verify docker-compose created a loadbalancer and two webservers.

```bash
docker-compose ps
```
	          Name                   Command          State                      Ports                    
	-----------------------------------------------------------------------------------------------------
	dockcpnginxlb_nginx-01_1   nginx -g daemon off;   Up      443/tcp, 80/tcp                             
	dockcpnginxlb_nginx-02_1   nginx -g daemon off;   Up      443/tcp, 80/tcp                             
	dockcpnginxlb_nginx-lb_1   nginx -g daemon off;   Up      0.0.0.0:4001->443/tcp, 0.0.0.0:4000->80/tcp


- View the docker images that were created

```bash
docker image ls
```
	REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
	nginx-lb            latest              627012489849        49 minutes ago      183 MB
	nginx               latest              5766334bdaa0        2 days ago          183 MB


- View the docker containers that were created

```bash
docker container ls
```
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                         NAMES
	f465648ebbf6        nginx-lb:latest     "nginx -g 'daemon ..."   8 minutes ago       Up 8 minutes        0.0.0.0:4000->80/tcp, 0.0.0.0:4001->443/tcp   dockcpnginxlb_nginx-lb_1
	e88784023544        nginx               "nginx -g 'daemon ..."   8 minutes ago       Up 8 minutes        80/tcp, 443/tcp                               dockcpnginxlb_nginx-01_1
	63c2d29dd0f7        nginx               "nginx -g 'daemon ..."   8 minutes ago       Up 8 minutes        80/tcp, 443/tcp                               dockcpnginxlb_nginx-02_1


- Test Docker networking. (nginx-lb wont start if the webservers are unreachable)

```bash
docker-compose run nginx-lb ping nginx-01
```
	PING nginx-01 (172.18.0.3): 56 data bytes
	64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.070 ms
	64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.086 ms
	64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.076 ms
	^C--- nginx-01 ping statistics ---
	3 packets transmitted, 3 packets received, 0% packet loss
	round-trip min/avg/max/stddev = 0.070/0.077/0.086/0.000 ms

```bash
docker-compose run nginx-lb ping nginx-02
```
	PING nginx-02 (172.18.0.2): 56 data bytes
	64 bytes from 172.18.0.2: icmp_seq=0 ttl=64 time=0.188 ms
	64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.085 ms
	64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.084 ms
	^C--- nginx-02 ping statistics ---
	3 packets transmitted, 3 packets received, 0% packet loss
	round-trip min/avg/max/stddev = 0.084/0.119/0.188/0.049 ms


## Test Application

### From any Web Browser - http://127.0.0.1:4000/

- View webserver logs

```bash
docker logs  dockcpnginxlb_nginx-01_1
```
	172.18.0.4 - - [09/Apr/2017:01:13:08 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36" "-"


```bash
docker logs  dockcpnginxlb_nginx-02_1
```
	172.18.0.4 - - [09/Apr/2017:01:20:28 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36" "-"


## Shut down and delete application

- Stop the application

```bash
docker-compose stop
```
	Stopping dockcpnginxlb_nginx-lb_1 ... done
	Stopping dockcpnginxlb_nginx-01_1 ... done
	Stopping dockcpnginxlb_nginx-02_1 ... done

- Completely remove the application (Docker images will not be removed)

```bash
docker-compose down --volumes
```
	Removing dockcpnginxlb_nginx-lb_run_2 ... done
	Removing dockcpnginxlb_nginx-lb_run_1 ... done
	Removing dockcpnginxlb_nginx-lb_1 ... done
	Removing dockcpnginxlb_nginx-01_1 ... done
	Removing dockcpnginxlb_nginx-02_1 ... done
	Removing network dockcpnginxlb_default


