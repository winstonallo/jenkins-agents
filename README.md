# Jenkins
## Basic Setup
`Dockerfile`
```Dockerfile
FROM jenkins/jenkins:latest

USER root

RUN apt-get update && apt-get install -y lsb-release python3-pip

RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg

RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list

RUN apt-get update && apt-get install -y docker-ce-cli

USER jenkins

RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```
### Build
```sh
docker build -t jenkins-blueocean .
```
### Create Jenkins Network
```sh
docker network create jenkins
```
### Run
```
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 jenkins-blueocean
```
### Get Admin Password
```sh
docker exec jenkins-blueocean cat /var/jenkins_home/initialAdminPassword
```
## Docker Cloud Setup
### Setup Worker
`alpine/socat` Container to forward traffic from Jenkins to Docker on host machine
```sh
docker run -d --restart=always --name jenkins-docker-proxy -p 127.0.0.1:2376:2375 --network jenkins -v /var/run/docker.sock:/var/run/docker
```
### Get IP Address for Worker Setup
```sh
docker inspect jenkins-docker-proxy | grep IPAddress
```
