### On Linux
- `lsb_release -a` to check the version of Linux
- docs.docker.com/engine/install/ubuntu
- minikube.sigs.k8s.io/docs/start

### Handy:
- `alias k=kubectl`

### Basics
- Cluster | instance of k8s (airport)
- Control pane | manages the cluster (airport control tower)
    - These are all actually containerised applications that run on a master node:
        - Kube API server | front end of control plane, with REST API (kubectl and kubeadm are CLI tools to interact with the API server via HTTP requests).
        - etcd | key value store (database) that stores the state of the cluster (e.g. number of nodes, pods, services, etc)
        - Scheduler | schedules pods to run on nodes (e.g. if a node is full, it will schedule the pod to run on another node)
        - Controller manager (basically a loop) | watches the state of the cluster and makes changes to the cluster to match the desired state (e.g. if a node goes down, it will create a new node to replace it)
        - Cloud controller manager | manages the cloud provider (e.g. AWS, Azure, GCP, etc)
- Worker nodes | run applications (planes)
    - Most K8s work with min. 3 nodes
    - Each node has:
        - Kubelet | agent that runs on each node and communicates with the control plane
        - Container runtime | software that runs containers (e.g. Docker, containerd, CRI-O, etc)
        - Kube-proxy | network proxy that runs on each node and maintains network rules on the node (e.g. load balancing, routing, etc)


- `minikube start` | create local cluster (in cloud use their k8s)
- `kubectl ... kubectl cluster-info`  | use local cluster
- `kubectl get nodes` | get nodes in cluster
- `kubectl get namespaces` | isolate and manage resources in namespaces
- `kubectl get pods -A` | get pods in all namespaces
- `kubectl get services -A`` | get services in all services (act as load balancer / direct traffic to pods)
- `minikube dashboard` | open dashboard in browser
- `minikube stop` | stop local cluster
- `minikube delete` | delete local cluster


### Create a namespace
- `kubectl apply -f namespace.yml` | create namespace as defined in in a YAML file

### Deployment an application
- `kubectl apply -f deployment.yaml` | create deployment as defined in a YAML file
- `kubectl get deployments -n <namespace>` | get deployments
- `kubectl get pods -n <namespace>` | get pods
- `kubectl delete pod <pod-name> -n <namespace>` | delete pod (will be recreated)

### Check health of pods
- `kubectl describe pod <pod-name> -n <namespace>` | check health of pod

### Check your application with busybox
- `kubectl get pods -n <namespace> -o wide`` | get a pods IP

- `kubectl apply -f busybox.yaml` | create busybox pod
- `kubectl exec -it <pod-name> -- /bin/sh` | execute shell in pod
- `wget -O- http://<service-name>:<port>` | check if service is running
- `cat index.html` | check if service is running


### View application logs
- `kubectl logs <pod-name> -n <namespace>` | view logs of pod
exit


### Quote Service (example App)
- `kubectl apply -f quote.yaml` | create quote service

**Use busybox to test application can accept traffic from inside the cluster**

```
# run a temporary busybox pod:
kubectl run -i --tty --rm debug --image=busybox --namespace=development -- sh

# from the busybox shell, test the service using wget:
wget -qO- http://quote-service:8080/
```

### Expose application to the outside world
A kubernetes service is an abstraction layer that exposes an application running on a set of pods as a network service (aka load balancer). 

Additionally there are three types of services:
- LoadBalancer | accessible from outside the cluster
- ClusterIP (default) | only accessible from within the cluster
- NodePort | accessible from outside the cluster

A load balance has a public (anyone can access) and static IP address (important as pod IPs can change frequently, so service IP needs to remain the same). The load balancer directs traffic to the pods.


##### In one tab
- `minikube tunnel` | create a tunnel to the load balancer

##### In another tab
- `kubectl apply -f service.yaml` | create service (this one is related to deployment.yaml as defined by app label `pod-info`) 
- `kubectl get services -n development` | get services

### Add resource requests and limits to a pod
- `kubectl apply -f deployment.yaml` | create deployment as defined in a YAML file

### Kubernetes Security
- ensure containers run as non-root users `securityContext: runAsNonRoot: true`
- ensure containers run with a read-only root filesystem `securityContext: readOnlyRootFilesystem: true`
- scan with synk (synk.io)


### Application Pods
You need to run an application that performs a one-time extract, transform, load (ETL) operation that transfers data from a SQL database to a data warehouse. What is the best way to run these application pods?

- StatefulSet
- DaemonSet
- Job
- Deployment

A Kubernetes Job will spin up a pod, run the container until its task is complete, and then terminate the pod. A Job is best for applications that perform one-time operations, like an ETL.
