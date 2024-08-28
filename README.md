# Kubernetes(k8s) - Minikube/kubectl command notes

In order to run the single node cluster we can use `minikube`. We need to install it locally and also it needs some virtualization tools/drivers eg: docker, virtualbox etc. Once we are done with docker and minikube installation we need to install `k8s cli` eg: `kubectl` to communiticate with k8s engine/cluster. We can install kubectl and minikube using `homebrew` formula eg: `brew install kubectl` and `brew install minikube`

Below is few components need to know about before start with k8s: 
- Cluster
- api server
- container runtime
- kube proxy
- etcd
- kubelet
- scheduler
- kube controller
- Nodes
- Pods
- Services
- Deployments

### What is a pod?
A single instance of a running process in a cluster.
It can run one or more containers and share the same resources.

There are mainly 2 types of nodes in k8s.
1. `master/control plane node`
2. `worker node`

### Below are the components of a `master node`
- **API SERVER**: The API server acts as the front door to the cluster, handling requests from clients (such as kubectl) and translating them into actions for other components.
- **Schedular**: Assign node to newly created Pods.
- **ETCD**: key-value store, having all cluster data
- **Control Manager**: Responsible for managing the state of the cluster.


### Below are  the components of a `worker node`
- **kubelet**: Agent, make sure containers running in pods.
- **Pods**: POD, container run in pod.
- **kube-proxy**: Maintains network rules for communication with pods.
- **container-runtime**: A tool responsible for running containers.


### Features of k8s
- Container Orchestration
- Scalability
- Load Balancing
- High Availability
- Rollouts & Rollback

### Start the minikube single node cluster
```bash
minikube start --driver=virtualbox # By default driver is docker engine
minikube status # Check the status of cluster
minikube dashboard # start the k8s dashboard in local server
minikube stop # Stop the minikube cluster
minikube delete # Delete the minikube cluster

kubectl cluster-info # Get the running cluster info
```

### Creating a pod/deployment

```bash
# Create a deployment using kubectl command

kubectl create deployment my-nginx --image=nginx:latest

# Above command will pull the nginx:latest docker image from docker hub and start the container in pod. We can now go to k8s dashboard and check there.

# Get deployments
kubectl get deployments

# Get pods
kubectl get pods

# Get nodes
kubectl get nodes

minikube dashboard

# Delete the deployments
kubectl delete deployment my-nignx
```
Now `nginx` is running on pod's port 80 but we can't directly access it since it is an isolated env. In order to access or port binding we need to create `services`
```bash
# Create a service using kubectl command

kubectl expose deployment my-nginx --port=80 --type=LoadBalancer

# Now we can check the services using below command
kubectl get services

# Now, we have created the service the but we need to tell it to minikube using below command

minikube service my-nginx

# Now we can access it on localhost
```