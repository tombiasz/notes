# Running docker containers
refs
- [Katakoda](https://www.katacoda.com/courses/kubernetes/kubectl-run-containers)
- [Kubernetes by example](http://kubernetesbyexample.com/)

## 1. start minikube
`minikube start`

## 2. checking nodes
the *nodes* are the worker machines where your pods run

`kubectl get node`

## 3. checking pods
`kubectl get pod`

a *pod* is a collection of containers sharing a network and mount namespace
and is the basic unit of deployment in Kubernetes. All containers in a pod
are scheduled on the same node.

## 4. first deploy
`kubectl run NAME --image=DOCKER_IMAGE_IN_REGISTRY --replicas=1`
`kubectl get deployments`
`kubectl describe deployment NAME`

a *deployment* is a supervisor for pods and replica sets,

## 5. exposing a port
after deploy
`kubectl expose deployment NAME --external-ip="172.17.0.52" --port=8000 --target-port=80`

single-command: deploy and expose
`kubectl run NAME2 --image=DOCKER_IMG_IN_REPO --replicas=1 --port=80 --hostport=8001`
(exposes the Pod via Docker Port Mapping)

## 6. pause containers
`docker ps | grep pause`

the *pause* container is part of each pod that is responsible to create shared
network, assign ip address within the pod for all business containers inside
this pod, also the pause container shared the volume for entire pod

## 7. scaling pods
`kubectl scale --replicas=3 deployment NAME`
once each pod starts it will be added to the load balancer service

## 8. services
`kubectl get service`
a service is an abstraction for pods, providing a stable, virtual IP (VIP) address.
While pods may come and go, services allow clients to reliably connect to the
containers running in the pods, using the VIP
