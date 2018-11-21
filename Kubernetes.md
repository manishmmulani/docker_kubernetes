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


Initializing a Cluster
> kubeadm init --token=102952.1a7dd4cc8d1f4cc5 --kubernetes-version $(kubeadm version -o short)

> sudo cp /etc/kubernetes/admin.conf $HOME/
> sudo chown $(id -u):$(id -g) $HOME/admin.conf
> export KUBECONFIG=$HOME/admin.conf

Below lists the token. For new nodes to join the cluster, they should provide the right token
> kubeadmin token list

Server ip and port can be obtained from admin.conf file above
> kubeadmin join --token=102952.1a7dd4cc8d1f4cc5 172.17.0.138:6443



