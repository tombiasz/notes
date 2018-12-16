<!-- TOC -->

- [1. Running docker containers](#1-running-docker-containers)
  - [1.1. start container](#11-start-container)
  - [1.2. list containers](#12-list-containers)
  - [1.3. run interactive session](#13-run-interactive-session)
  - [1.4. Background process](#14-background-process)
  - [1.5. Connect to existing container](#15-connect-to-existing-container)
  - [1.6. List images](#16-list-images)
  - [1.7. Remove images](#17-remove-images)
  - [1.8. List containers](#18-list-containers)
  - [1.9. Remove containers](#19-remove-containers)
  - [1.10. Refs](#110-refs)
- [2. Running docker from Dockerfile](#2-running-docker-from-dockerfile)
  - [2.1. dockerfile](#21-dockerfile)
  - [2.2. build](#22-build)
  - [2.3. run container from image](#23-run-container-from-image)

<!-- /TOC -->


# 1. Running docker containers

## 1.1. start container
`docker container run alpine hostname`

* similar to `docker-compose run XXX hostname`
* run command `hostname ` inside container `alpine`
* `docker container run` exists as soon as process exists. This stops the container

## 1.2. list containers
`docker container ls —all`
* list containers and their status eg `exit(0)`
* similar to `docker-compose ps`

## 1.3. run interactive session
`docker container  —interactive —tty —rm run ubuntu bash`

* Similar to: `docker-compose run XXX bash`
* `—rm`will remove docker at the end of the session. It won’t be visible when listing (ad. 2)
* `-it` run interactive tty session

## 1.4. Background process
```
docker container run \
 --detach \
 --name mydb \
 -e MYSQL_ROOT_PASSWORD=my-secret-pw \
 mysql:latest
```

## 1.5. Connect to existing container
`docker exec -it mydb mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version`

* Similar to `docker-compose run XXX bash`
* `docker exec` == `docker container exec`

## 1.6. List images
` docker image ls --all`

## 1.7. Remove images
`docker image rm IMAGE_NAME`
`docker image prune`
`docker image prune --all`

## 1.8. List containers
`docker container ls --all`

## 1.9. Remove containers
`docker container rm CONTAINER_NAME`
`docker container prune`
`docker container prune --all`

## 1.10. Refs
- [Docker for Beginners - Linux](https://training.play-with-docker.com/beginner-linux/)


# 2. Running docker from Dockerfile

## 2.1. dockerfile
```
// docker/Dockerfile

FROM node:10.13.0-slim

WORKDIR /app

COPY . .
```

## 2.2. build
`docker build -f docker/Dockerfile -t IMG_NAME .`

build image can be checked with
`docker image ls`

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
IMG_NAME            latest              83768f672e73        7 minutes ago       184MB
```

## 2.3. run container from image
`docker container run -it --rm IMG_NAME bash`
