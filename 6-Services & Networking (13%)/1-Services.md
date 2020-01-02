# Services

- Deployments make it easy to create a set of replica pods that can be dynamically scaled, updated, and replaced. However, providing network access to those pods for other components is difficult. Services provide a layer of abstraction that solves this problem. Clients can simply access the service, which dynamically proxies traffic to the current set of replicas. In this lesson, we will discuss services and demonstrate how to create one that exposes a deployment's replica pods.

- Relevant Documentation
https://kubernetes.io/docs/concepts/services-networking/service/
https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

- Lesson Reference

- Create a deployment:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

- Expose the deployment's replica pods with a service:

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80

- You can get more information about the service with these commands:

`kubectl get svc`
`kubectl get endpoints my-service`

# lesson - Services

`Services:` services create an abstraction layer which provides network access to a dynamic set of pods. 

- Let us imagine that we have 3 pods. These pods could be 3 replicas from a deployment. They don't have to be, but that is the most common use case with services that these are replica pods that are part of a deployment. 

- Now I want to access whatever application these pods are running. So if I have 3 different replica pods, the question then becomes how do I access these pods? Now? I could obtain the IP address for the 1st pod and just access that pod directly. 

- But the problem is these set of pods could change. I could add replicas, remove replicas, replace replicas, their IP addresses are always gonna change potentially. Even the number of replicas could change. 

- So the dynamic nature of my set of pods makes it very difficult to try to access the pods directly. 

- So we can add a service as an abstraction layer on top of our list of pods, and that way we just have to access the service directly because this service going to handle the process of actually communicating with this dynamic set of pods on the backend. 

- So anything that needs to access whatever application is being run in a set of replica pods doesn't have to worry about where the replicas are or what their IP addresses are, or even how many replicas there are. All we have to do is access the service. 

- Most services use a selector in order to determine which pods they're going to forward traffic to. 

- We have talked about selectors in some of the earlier lessons. But we're going to see selectors again when we talk about services so the service provides a selector and anything that tries to access the service with network traffic, that traffic is going to proxy to one of the pods that is selected through that selector.
 
# Explanation 

- I'm going to create a deployment. We're just going to use that in order to test our service. 

`vi nginx-deployment.yml`

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80


- This is just a simple deployment and you'll notice we're creating some pods with the label app equals nginx and they're just simple pods running nginx and exposing Port 80. 
- `kubectl apply -f nginx-deployment.yml` >> deployment is now created , and that should create 3 replica pods.  

- So what I'm gonna do is I'm going to create a service that will expose these 3 replica pods. 

`vi my-service.yml`

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 80


- I'll do my `API version`, which is `v1`. that `kind` will be `service`. 
- And then I will give it a `name` of `my-service` and then I can provide the service specifications. 
- The first thing I'm gonna do is I'm going to set the type. 

- There are multiple different types of services, and we'll talk a little bit more about that. In just a second. I'm going to set this to `ClusterIP` service, which is actually the `default`. 

1- If you want to build a service that is designed to be accessed by other pods in the cluster then ClusterIP is the type that you want to use, and it's probably the most common service type to use in an actual production scenario. 

- Then I'm going to add a `selector`. You may remember that we talked about selectors a moment ago and how we use those in our service to determine which pods are going to be part of the service. And you might remember, in the deployment we had, this `app equals nginx` label being attached to all the pods. 

- So we want to expose those pods. So we're going to use this selector to select those pods that we want to expose with the service.
 
- Also I need to specify ports, so I only need one port, and I'm going to specify the TCP protocol for that port, and I'll just set the actual port to a 8080. This is the port that service is going to listen on. And it doesn't necessarily have to be the same port that the containers in the pods are listening on. So we can actually map a port here to a completely different port on the actual backend pods. So just to show you how that works, I'm going to use a different port you may remember, based on the deployment specifications our pods are actually listening on port 80. But the service is going to listen on Port 8080. 

- So anything that needs to access the service will use that port 8080 and then the service itself will proxy that traffic to the pods on the correct port on the backend and in order to get that to work, we need to specify a targetPort. And this is the port where the actual pods are listening. So our nginx containers are pods are listening on that container port of 80. So that's gonna be our target port. 

`kubectl apply -f my-service.yml` Service is now created. 

`kubectl get svc` Here's my service right here. We can see that it has been assigned an IP address in the cluster. And there's that port that it's listening on 8080. 

`kubectl get ep my-service` or `kubectl get endpoints my-service` >>  This is going to get a list of the end points that are associated with that service. 

- So the endpoint represents the connection between the service and the actual pods. So we have 3 pod replicas and therefore we have 3 endpoints listed here and we can see it has the IP address of each pod and also that targetPort, which is the port where the pod itself is actually listening, which is Port 80 in our case. 

- So it looks like our service has been successfully created and it has 3 endpoints. So it has successfully selected our 3 pods and it's properly map to the pods that we expect it to be mapped too. 

- So if you remember in our service we had our type of ClusterIP. And there are 4 different types of services that behave in different ways. 


ClusterIP and NodePort are probably the most important to have knowledge of when it comes to the CKAD exam. 

`ClusterIP`: it exposes the service using an IP internal to the clusters. So if you want to access the service from pods or other entities within the cluster, then cluster IP is what you want to use. 

`NodePort`: it exposes the service outside the cluster. NodePort is commonly used for testing purposes and what it does is it actually selects an open port on all the nodes and listens on that port on each, each one of the nodes, so you can actually access the service externally by accessing that port on any of the actual kubernetes cluster nodes. 

`LoadBalancer` It automatically creates loadbalancers, for example, in AWS or on Google Cloud platform. So if you're running your kubernetes cluster on a cloud platform, you could easily create a loadbalancer. To be able to access your services using the loadbalancer type, but that only works if you're actually running within a cloud platform or you're set up to interact with a cloud platform. So the clusters that we have been working with up to this point are configured that way. So loadbalancer services are not going to work. 

`ExternalName` This is a little bit different from all the others. Instead of mapping, services to actual pods within the cluster. The ExternalName service exposes things that are external to the cluster, and it basically just provides that additional layer of abstraction between things inside your cluster that want to access a service and some type of application or entity that lives outside the cluster. 

Now you don't really need to worry too much about loadbalancer or ExternalName, but make sure you at least have a general understanding of what the cluster IP and the NodePort Service is doing. 

So we've talked about Service's. We've talked about how we can use services to provide a networking abstraction layer on top of a dynamic set of pods, And I've also shown you how to create a service in order to expose some replica pods that are part of a deployment. So now you have a basic understanding of services and how to create them.