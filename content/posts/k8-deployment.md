---
title: "K8 Service & Deployment"
date: 2023-02-09T11:54:25+01:00
draft: false
---

In [this](https://www.bjornvandijkman.com/posts/docker/) blog I talked about dockerizing a machine learning application. This article will be about deploying this application onto a Kubernetes cluster. Initially we will need to create two types of resources: a deployment and a service.

### The deployment
A deployment describes the desired state for Pods and Replicasets. 👇 Down below you can see the deployment file for my FASTAPI application. 

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: fastapi-deployment

spec:
  replicas: 2
  selector:
    matchLabels:
      app: fastapi

  template:
    metadata:
      labels:
        app: fastapi
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 2000
        fsGroup: 3000
      containers:
      - name: fastapi-container
        image: app:latest
        imagePullPolicy: Never
        resources:
          limits:
            memory: 512Mi
            cpu: "1"
          requests:
            memory: 256Mi
            cpu: "0.2"
        ports:
        - containerPort: 8000 # Same as service target port
```

It creates a replicaset of three FASTAPI pods, meaning that there will be 3️⃣ pods that contain our app container. I have included **securityContext** in **spec** as we do not want to run the container as the root user:
- runAsUser: Specifies the user ID that the container should run as. This is an important security setting, as it restricts the access that the container has to the host file system and other resources.
- runAsGroup: Specifies the group ID that the container should run as. Like runAsUser, this setting is used to restrict the access that the container has to resources.
- fsGroup: Specifies the group ID that should own the files created by the container. This setting is used to ensure that files created by the container have the correct ownership and permissions.

I am using:
```yaml
imagePullPolicy: Never
```
as I am currently deploying local docker images that we do not need to pull from [docker hub](https://hub.docker.com). Port 8000 is the port which is open for communication between the containers and the service resource, which we will get to now. ⬇️


### The service
This is an abstract way of exposing an application that is running on a set of Pods as a network service. 🤔 Can't we just expose the IP addresses of the pods itself? Well, no. The pods are nonpermanent resources, they can be created and destroyed, and will match the desired state of your deployment. Each pod does have its own IP address, but if a pot gets destroyed and recreated, it will have a new IP address. Communicating using the IP addresses of the pods is therefore not durable, and we need to connect applications in a different way. 

💡 Enter the service resource. It defines a policy by which to access the pods. The set of pods to access by the service is usually determined by a **selector**. This abstraction enables the decoupling between different pods, meaning that they do not to keep track of each other in order to communicate.  

```yaml
apiVersion: v1
kind: Service

metadata:
  name: fastapi-service

spec:
  type: LoadBalancer
  ports:
    - protocol: TCP
      name: service-port
      targetPort: 8000 # MUST be the port exposed by FastAPI "CMD" in DockerFile
      port: 8000 # Can be any port
  selector:
    app: fastapi
```

🙉 **Port** specifies the port number that the service should listen on. Clients will access the service on this port. The **targetPort** specifies the port number that the service should forward incoming traffic to. Note: the traffic is only forwarded to port 8000 on the pods that are selected by the **selector** (fastapi). 

The protocol TCP specifies the way that the service should communicate 📡. One alternative is to TCP is User Datagram Protocol (UDP), which is suitable for a service when you want fast, low-overhead communication and can tolerate some loss of data. If you need guaranteed delivery of messages, you would use **TCP** as the protocol instead.   

## Up and running🏃‍♂️

We can apply the service and deployment by running 
```bash
kubectl apply -f api-service.yaml
kubectl apply -f api-deployment.yaml
```

To be able to send requests to our FASTAPI container, we need to port-forward the service port to our local host:
```bash
kubectl port-forward service/fastapi-service 8000:8000
```

We can then go to http://localhost:8000 in your browser, where I can see the following message:
```json
{"message":"API is up and running!"}
```

As that is my root endpoint in the FASTAPI app:
```python
@app.get("/")
def read_root():
    """
    A simple endpoint to check if the API is running.
    """
    return {"message": "API is up and running!"}
```