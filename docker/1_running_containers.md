# Running docker containers
Base on [Docker for Beginners - Linux](https://training.play-with-docker.com/beginner-linux/)

## 1. start container
`docker container run alpine hostname` 

* similar to `docker-compose run XXX hostname`
* run command `hostname ` inside container `alpine`
* `docker container run` exists as soon as process exists. This stops the container

## 2. list containers
`docker container ls —all` list containers and their status eg `exit(0)`
Similar to `docker-compose ps`

## 3. run interactive session
`docker container  —interactive —tty —rm run ubuntu bash`

* Similar to: `docker-compose run XXX bash`
* `—rm`will remove docker at the end of the session. It won’t be visible when listing (ad. 2)
* `-it` run interactive tty session

## 4. Background process
```
docker container run \
 --detach \
 --name mydb \
 -e MYSQL_ROOT_PASSWORD=my-secret-pw \
 mysql:latest
```

## 5. Connect to existing container
`docker exec -it mydb mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version`

* Similar to `docker-compose run XXX bash`
* `docker exec` == `docker container exec`

## 6. List images
` docker image ls`



