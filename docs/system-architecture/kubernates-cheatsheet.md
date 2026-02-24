# Kubernetes Commands

- Kubernetes is a platform where you can run multiple instance of docker images or containers

- Kubernetes is a tool for managing containerized applications across clusters. It provides basic mechanisms for deployment, maintenance, and scaling of application containers across clusters

- Kubernetes runs on top of Docker Engine. It uses it as its underlying infrastructure layer.

## Kubernetes Cluster

    A collection of nodes + a master to manage them

## Node

    A virtual Machine that will run our containers

## Pod

    - More or less a running a container
    - Technically, a pod can run multiple container

## Deployment

    Monitors a set of pods, make sure they are running and restarts them if they crash

## Service

    Provides an easy-to-remember URL to access a running container

### Docker Image

- Single file with all the deps and config required to run a program

### Docker Container

- Instance of an program or an image

## Kubernetes Steps

### Create a POD

```yaml
# K8s is extensible - we can add in out own custom objects. This specifies the set of objects we want K8s to look at
apiVersion: v1

# The type of object we want to create
kind: Pod

#  Config Options for the object we are about to create
metadata:
  name: posts # When a Pod is created, give it a name of "posts"

# The exact attributes we want to apply to the object we are about to create
spec:
  # We can create many containers in a single Pod
  containers:
    - name: posts # Make a container with a name of "posts"
      image: debirapid/posts:0.0.1 # The exact image we want to use
```

### (Recommended)Create a Deployment and then Create Pods

## Kubernetes Commands and Description

| Command                                                 | Description                                                   | Example                                                   |
| :------------------------------------------------------ | :------------------------------------------------------------ | --------------------------------------------------------- |
| kubectl version                                         | Version info of Kubernetes                                    |                                                           |
| kubectl apply -f \<config-file\>                        | Create a Pod in Kubernetes                                    | kubectl apply -f posts.yaml                               |
| kubectl get pods                                        | Get all the running pods in the Kubernetes                    |                                                           |
| kubectl exec [pod_name] -it [cmd]                       | Execute the given command in a running pod                    | kubectl exec posts -it -- sh                              |
| kubectl logs [pod_name]                                 | Print the logs from the given pod                             |                                                           |
| kubectl logs --follow [pod_name]                        | Get the live logs from the pod                                |                                                           |
| kubectl delete pod [pod_name]                           | Deletes a given pod                                           | kubectl delete pod posts                                  |
| kubectl describe pod [pod_name]                         | Print out some information about the running pod              |                                                           |
| kubectl get deployments                                 | List all the running deployments                              |                                                           |
| kubectl describe deployment [depl_name]                 | Print out details about a specific deployment                 |                                                           |
| kubectl apply -f [config_file]                          | Create adeployment out of a config file                       | kubectl apply -f posts-depl.yaml                          |
| kubectl delete deployment [depl_name]                   | Delete a deployment                                           |                                                           |
| kubectl get services                                    | Get all kinds of services, [ClusterIP/NodePort/Load Balancer] |                                                           |
| kubectl describe service [srv_name]                     | Print out details about a specific service                    |                                                           |
| kubectl rollout restart deployment [depl_name]          | restart a deployment or a specific set of resources           |                                                           |
| kubectl port-forward [pod_name] [local_port]:[pod_port] | set up a port forwarding from local to Kubernetes pod's port  | kubectl port-forward nats-depl-7cd6497c7f-6xgsx 4222:4222 |

## Networking with Services

- To be able to access the pod from outside or make inter pod communication(microservices) we need to create services.
- There are several kinds of services.

### Cluster IP (default)

    Sets up an easy to remember URL to access a pod, Only exposes pods in the cluster.

### Node Port

    Makes a pod accessible from outside the cluster. Usually only used for dev purposes.

### Load Balancer

    Makes a pod accessible from outside the cluster. This is the right way to expose a pod to the outside world.

### External Name

    Redirects an in-cluster request to a CNAME url.

# Load Balancer

- A load balancer is responsible for distributing incoming traffic across multiple backend servers or instances.

- Load balancers can be created using cloud providers or by installing software such as nginx ingress controller on your own servers.
