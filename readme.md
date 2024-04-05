## YouTube Link

For the full 1 hour course watch on youtube:
https://www.youtube.com/watch?v=6YZvp2GwT0A

# Installation

## Build the Jenkins BlueOcean Docker Image (or pull and use the one I built)

```
docker build -t myjenkins-blueocean:2.440.2-1 .
```

## Create the network 'jenkins'

```
docker network create jenkins
```

## Run the Container

### MacOS / Linux

```
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.440.2-1
```

### Windows

```
docker run --name jenkins-blueocean --restart=on-failure --detach `
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 `
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
  --volume jenkins-data:/var/jenkins_home `
  --volume jenkins-docker-certs:/certs/client:ro `
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.440.2-1
```

## Get the Password

```
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

## Connect to the Jenkins

```
http://localhost:8080/
```

## Installation Reference:

https://www.jenkins.io/doc/book/installing/docker/

## alpine/socat container to forward traffic from Jenkins to Docker Desktop on Host Machine

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin
下面命令的作用是在容器内运行 socat，将本地的 TCP 请求转发到 Docker 守护进程的 Unix 套接字上。这样，你就可以在容器外部通过 TCP 协议连接 Docker 守护进程。这样就可以创建新的 Agents 用来分配 build 任务，从而 jenkins master 只负责分发任务。这里是 cloud（Docker） 类型的 nodes。现代企业一般用 permanent agent（比如 windows，linux server）的少一些。

_socat 在 docker 每次重启 ip 不一样，需要重启后更新 jekins 的 cloud node_

```
docker run -d --restart=always -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker.sock alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock

docker inspect <container_id> | grep IPAddress
```

## Using my Jenkins Python Agent

_Default jenkins/agent:latest-alpine has no python added, so use Dockerfile-alpine-custom to build a customized agent image._

```
docker pull guodongdocker/jenkins-agent-alpine-custom
```
