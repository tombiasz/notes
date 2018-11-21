# Deploy with minikube
refs
- [Katakoda](https://www.katacoda.com/courses/kubernetes/creating-kubernetes-yaml-definitions)

## prerequisites
Install minkube
[minikube](https://kubernetes.io/docs/setup/minikube/#installation)

## 1. deployment
runs docker image on docker cluster. App is accessible from minikube host. You
can ssh into it and ping/curl the image

```
// deployment.yaml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: APP_NAME
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: APP_NAME
    spec:
      containers:
      - name: APP_NAME
        image: DOCKER_IMG_IN_REPOSITORY
        ports:
        - containerPort: 80
```
`kubectl create -f deployment.yaml`
`kubectl get deployment`
`kubectl describe deployment APP_NAME`

## 2. service
create service that will expose port to the deployment.

```
// service.yaml

apiVersion: v1
kind: Service
metadata:
  name: SVC_NAME
  labels:
    app: APP_NAME
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: APP_NAME
```

`kubectl create -f service.yaml`
`kubectl get svc`
`kubectl describe svc SVC_NAME`

## 3. scaling
```
// deployment.yml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: APP_NAME
spec:
  replicas: 100 // <-- update
  template:
    metadata:
      labels:
        app: APP_NAME
    spec:
      containers:
      - name: APP_NAME
        image: DOCKER_IMG_IN_REPOSITORY
        ports:
        - containerPort: 80
```

`kubectl apply -f deployment.yaml`

## Notes
- ssh into minikube `minikube ssh`
- get external address to service `minikube service APP_NAME --url`
