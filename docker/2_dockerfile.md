# Running docker from Dockerfile

## 1. dockerfile
```
// docker/Dockerfile

FROM node:10.13.0-slim

WORKDIR /app

COPY . .
```

## 2. build
`docker build -f docker/Dockerfile -t IMG_NAME .`

build image can be checked with
`docker image ls`

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
IMG_NAME            latest              83768f672e73        7 minutes ago       184MB
```

## 3. run container from image
`docker container run -it --rm IMG_NAME bash`
