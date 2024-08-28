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
# Now we can check the services using below command. Here port 80 is application port running inside the container.

kubectl get services

# Now, we have created the service the but we need to tell it to minikube using below command

minikube service my-nginx

# Now we can access it on localhost
```

Getting the logs of a particular Pod
```bash
kubectl get pods # Get the name of the pod

kubectl logs <pod-name> # we can also -f flag to stream the log

# Getting more information of pods
kubectl describe pods

```

k8s rollouts.
Updating the deployments.

```bash
kubectl get deployments

kubectl get pods

kubectl set image deployment <deploymentname> <containername>=<docker image with new tag>
# In above command get the container name from minikube dashboard. Here we are replacing the exiting docker image with different version. 

kubectl get pods
# You can notice that it is creating a new container for new version of image. It will not remove the previous deployment/container unless newly created deployment is up and running. Once newly created pod is up and running then it will delete the old pod and we don't need to make any changes in services and all.
```

k8s rollback.
Let's say we are updating a deployment with new docker image version that really doesn't exists. So it will try to pull the image indefinetely. And it will not close/delete the old pod, your old pod is still up and running. 

```bash
kubectl set image deployment <deployment-name> <container-name>=<docker image with new tag>

kubectl get pods

# Now check the rollout status(current status) with below command
kubectl rollout status deployment <deployment-name>

# Now in order to rollback the deployment we hit the below command
kubectl rollout undo deployment <deployment-name>

kubectl get pods
```

Now, think a scenario: Your app is running inside a k8s pod and something goes wrong with your code or any unexpected error happens and your app is crashed and pod is down. So k8s automatically restarts your pod and this is called `self healing`. You can get an idea when you run `kubectl get pods` there is one column `restarts` which tells how many times it restarted.

But still there is minor down-time since we have only one replica. If we don't want any downtime or if we want to load balance our pods then we can create multiple replica of our deployment on the fly using below command.
```bash
cubectl scale deployment <deployment-name> --replicas=4

# Now check the number of pods
cubectl get pods

# Now if need it back to 1 replicas; run the above command again with 1 replicas and it will automatically delete other pods.
```


### Now We can do the above operation using YAML configuration.
Create a yaml file. You can give any name to yaml file
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    # Unique key of the Deployment instance
    name: my-node-app
spec:
    # 2 Pods should exist at all times.
    replicas: 2
    selector:
        matchLabels:
            app: node-app
    template:
        metadata:
            labels:
                # Apply this label to pods and default
                # the Deployment label selector to this value
                app: node-app
        spec:
            containers:
            - name: node-app
              # Run this image
              image: <docker-image-name-with-tags>       
```
Now run the YAML file using below `kubectl` command
```bash
kubectl apply -f <yaml-file-name.yaml>

# Now check the pods
kubectl get pods
```

Now if we make any changes in the YAML file then we only need to run the above command again and again. So easy.

Now in order to access these pods locally/localhost we have to create a service.
```yaml
apiVersion: apps/v1
kind: Service
metadata:
    # Unique key of the service instance
    name: service-my-node-app
spec:
    ports:
        # Accept traffic sent to port 8080
        - name: http
        port: 8080
        targetPort: 3000
    selector:
        # Loadbalance traffic across Pods matching
        # this label selector
        app: node-app
    # Create an HA poroxy in the cloud provider
    # with an External IP address - *Only supported
    # by some cloud providers*
    type: LoadBalancer    
```

Now check the already existing services using below command.
`kubectl get services` and you can delete any service using this command `kubectl delete service node-app`

Now Use the same command to create this service.
```bash
kubectl apply -f <yaml-service-file.yaml>

minikube service service-my-node-app

kubectl get pods
```

Now in order to delete/terminate deployments; run below command
```bash
kubectl delete -f <yaml-deployment-file-name.yaml>

kubectl get deployments
kubectl get pods
```
