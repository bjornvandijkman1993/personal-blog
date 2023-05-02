---
title: "K8s Pods"
date: 2023-05-02T11:07:51+02:00
draft: false
---

### Introduction
A Kubernetes pod is a group of **one or more containers** that share storage, network resources, and a unique IP address. Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. They are used to run a single instance of an application in Kubernetes. 

### Why use pods?
So what are some of the benefits of using pods compared to running containers directly on a node?

1. **Multi-container management:** a common use case for pods is to run multiple containers that are tightly coupled together. An example of this is a primary application in one container and another container that has logging or monitoring functionalities. Communication between these containers is done via localhost, as the containers share the same network resources.
2. **Better fault tolerance:** Kubernetes pods are also used to provide fault tolerance for your applications. If a pod crashes, Kubernetes will automatically create a new pod to replace it. This ensures that your application is always available, even if one of the pods crashes.
3. **Health checks and self-healing:** Kubernetes can perform health checks on your pods and restart them if they fail. This ensures that your application is always running and available to users.
4. **Easier scaling:** Pods make it easier to scale your applications both horizontally and vertically, as Kubernetes can manage the process of adding or removing replicas of your pods based on demand. 
5. **Resource management:** Kubernetes can manage the resources used by your pods, such as CPU and memory. This ensures that your application has enough resources to run properly.

### Where in the architecture do pods fit?
Here is a brief overview of how Pods fit into the Kubernetes architecture:
1. **Node:** A Node is a worker machine in a Kubernetes cluster, which can be either a physical or a virtual machine. Each Node runs one or more pods. 
2. **Control plane:** The control plane is responsible for managing the state of the cluster, including scheduling pods on nodes and monitoring their health. It consists of several components, including the API server, scheduler, and controller manager.
3. **Scheduler**: This is responsible for scheduling pods on nodes based on resource requirements and other constraints.
4. **API server:** This is used to create, update and manage Kubernetes objects, including pods. 
5. **Controller manager:** This is responsible for managing the state of the cluster, including scheduling pods on nodes and monitoring their health.
6. **etcd:** This is a distributed key-value store that stores the state of the cluster. It is used by the API server to store and retrieve information about pods, nodes, and other Kubernetes objects.
7. **Services**: Services are used to expose pods to the outside world. They provide a stable IP address and DNS name for pods, so that other pods can communicate with them.

### How to create and manage pods?
Let's take a look at how you can create and manage pods in Kubernetes. We'll create a file named `nginx-pod.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

- An api version is included, as there are different versions of the Kubernetes API containing different features and behaviors.
- In Metadata we can store additional information about an object, such as its name, namespace, labels, annotations and creation timestamp.  
- The spec section contains the desired state of the object, which is the state that Kubernetes will try to achieve. In this case, we want to create a pod with a single container running the `nginx` web server. Furthermore, we want to expose port 80 of the container to the outside world.

Let's save the file and create the pod using the "kubectl apply" command:

```bash
kubectl apply -f nginx-pod.yaml
```

This will create a pod with a single container running the nginx web server.

To verify that the pod has been created, use the "kubectl get pods" command, which will list all pods in your cluster:

```bash
kubectl get pods
```
You should see the nginx pod in the output:

```bash
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          2m
```

To get more information about the Pod, run:

```bash
kubectl describe pod simple-pod
```

This will show you detailed information about the pod, including the pod's IP address, the node it's running on, and the containers running inside the pod.

To get the logs from the nginx container running inside the pod, use the "kubectl logs" command:

```bash
kubectl logs nginx-pod
```

This will print the logs from the nginx container to your terminal.

Nginx is now running inside a pod in your Kubernetes cluster. But how can you access it? To access the nginx web server running inside the pod, you can use the "kubectl port-forward" command like this:

```bash
kubectl port-forward nginx-pod 8080:80
```

This will forward port 8080 on your local machine to port 80 on the nginx pod. Now you can access the nginx web server by opening http://localhost:8080 in your browser.

I mentioned that pods allow you to run multiple containers within a single pod. They share the same network and storage, and can communicate with each other using localhost. Here's an example of a multi-container Pod YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: sidecar-container
    image: busybox
    command: ['sh', '-c', 'echo "Hello from the sidecar container!" && sleep 3600']
```

It is also possible to create init containers, which are containers that run before the main container in a Pod. A common use case for init containers is to perform some initialization tasks before the main container starts. For example, you can use an init container to download a configuration file from a remote server and store it in a shared volume. The main container can then read the configuration file from the shared volume. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  initContainers:
  - name: init-container
    image: busybox
    command: ['sh', '-c', 'echo "Initializing the Pod..." && sleep 5']
  containers:
  - name: main-container
    image: nginx:latest
    ports:
    - containerPort: 80****
```

Finally, if you need to delete a pod, use the `kubectl delete` command:
    
```bash
kubectl delete pod nginx-pod
```

This command will remove the specified pod from your cluster.


