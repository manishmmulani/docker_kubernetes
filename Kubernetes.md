minikube creates a VM and single-node kubernetes cluster is running within that VM
> minikube start

> kubectl cluster-info

> kubectl get nodes
>> Status of node = READY means it's ready to accept applications to be deployed

Deploy an app

> kubectl run first-deployment --image=katacoda/docker-http-server --port=80
>> deployment.apps/first-deployment created

Status of the deployment can be discovered via running pods

> kubectl get pods

Expose deployed container via different options eg: type=NodePort that provides dynamic port to the container

> kubectl expose deployment first-deployment --port=80 --type=NodePort

To know the port through which container is exposed

> kubectl get svc first-deployment

> curl http://host01:31556
>> Returns the host that processed the request


