# Liveness and Readiness Probes

* Kubernetes is often able to detect problems with containers and respond appropriately without the need for specialized configuration. But sometimes we need additional control over how Kubernetes determines container status. Kubernetes probes provide the ability to customize how Kubernetes detects the status of containers, allowing us to build more sophisticated mechanisms for managing container health. In this lesson, we discuss liveness and readiness probes in Kubernetes, and demonstrate how to create and configure them.

- Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/


- Lesson Reference

- Here is a pod with a liveness probe that uses a command:

`vim my-liveness-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: my-liveness-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    livenessProbe:
      exec:
        command:
        - echo
        - testing
      initialDelaySeconds: 5
      periodSeconds: 5
      
- Here is a pod with a readiness probe that uses an http request:

`vim my-readiness-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: my-readiness-pod
spec:
  containers:
  - name: myapp-container
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5


- lesson-Liveness and Readiness Probes
---------------------------------------

- `Probes` : Allow you to customize how kubernetes determines the current status of your containers. 

- `Liveness probes`: indicates whether or not the container is running properly, and it governs when the cluster will automatically stop or restart the container. So if a container crashes or stops running, kubernetes is automatically going to detect that, and it will restart the container. 

- But sometimes you need a little bit more customization than that. You may need to detect whether or not your container is healthy in a more complex or sophisticated way, and that is what Liveness probes allow you to do. 

- They essentially allow you to provide your own custom criteria to determine whether or not a container is healthy and if the container becomes unhealthy, it will be restarted. 

- `Readiness probes`: They're similar to Liveness probes. But while Liveness probes detect the current health status of the pod, a readiness probe detects whether or not the pod is ready to receive requests and governs whether requests will be forwarded to the pod.

- A lot of containers take a little bit of time to start up, and even once the container is running, it may take a little bit of time before the container is actually ready to start processing requests from users. And that's where the readiness probe comes in. 

- It allows you to provide some custom criteria to determine whether or not the container is ready to service requests, and until the container becomes ready, the pod will not receive requests from users. So if you're running multiple replicas of container within your cluster, only the ones that are ready will process requests, and it will wait until the containers ready before it starts sending requests to that container.
 
- There are different ways for Liveness and readiness probes to do. Checked the status of the container. They can do things like `run a command` inside the container and interpret the output. Or they can `make http requests` within the pod in order to determine the status of those containers. 

- Now let's create a Liveness probe. As you can see over here on the right. It's really just a matter of adding a Liveness probe to our container spec. 

`vim my-liveness-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: my-liveness-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    livenessProbe:
      exec:
        command:
        - echo
        - testing
      initialDelaySeconds: 5
      periodSeconds: 5
      

- Let's create a sample pod. this is called my-liveness-pod. And I've got some basic yaml configuration here and add a liveness probe.
- We just need to add this liveness probe attributes to our container spec. 

- There are many different kinds of liveness probes.

1- Exec probe: This allows you to run a command to determine the result of the probe. 

`Example`

- We will just do a command and I'm gonna list a list of the parts of the command here, and all I'm doing is I'm just echoing this string right here, testing to the console. So essentially, what this liveness probe is doing is it's just making sure that the consul within the container is responsive to commands. 

- If for some reason we're unable to run this command and it's unresponsive, then the liveness probe will fail and the container will be restarted.
 
- We will add a couple of additional attributes here.

1- initialDelaySeconds attribute : when I fire up this pod, the liveness probe is going to wait this number of seconds before it runs the liveness probe for the first time. 

- So if you have a pod that might take a couple seconds before you're actually ready to run the liveness probe, you can use this to delay that initial start of the liveness Probe. 

2- periodSeconds:  set it to 5, this means this probe is going to run every five seconds. 
 
`kubectl apply -f my-liveness-pod.yml`

`kubectl get pods` >>  We can see that my-liveness-pod is currently in the running status. 

- So that means my-liveness-pod is working. If it wasn't working, then the status would not be running in. The pod would be restarting. 




- Now we create a readiness probe. So I'll just make a deployment, descriptor. And I'll just call it my-readiness-pod.yml, No. And I'll put in a basic pod descriptor. I'm gonna use nginx this time because I want to show you how to do a http request probe.
 
`vim my-readiness-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: my-readiness-pod
spec:
  containers:
  - name: myapp-container
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5 
 
- Last time we used a command. This time we'll use an http request. So to create a readiness probe. I'll just add the readinessProbe key and then for the type instead of doing an exec which would allow me to run a command. I'm going to do an httpGet so this readiness probe is actually gonna make an http request to the pod. And the response of that http request is going to determine the status of the probe. 

`httpGet key` 

1- `path:` This is the path that it's gonna make the request to, and I'll just put / So we'll just hit the root and point to our nginx server. 

2- `port:` I'll just specify the default port of 80. So what this probe is gonna do is it's gonna make a request to the pod on Port 80 which is going to get a response from this nginx container. 

- Now if that request succeeds, then the container will be considered ready to receive requests. 
- I can also put in :
 
1- `initialDelaySeconds:` it determine the initial delay before it starts running the probe
 
2- `periodSeconds:` it determines how often it runs the probe. 

- So this is gonna wait five seconds before it runs the http request and then every five seconds after that, it's going to run it again. 

`kubectl apply -f my-readiness-pod.yml` >> apply the pod. 

`kubectl get pods` >> the pod is running. 

- Now, if a readiness probe has not yet succeeded, you'll see that reflected under the ready column here. So it says one out of one ready That means that my nginx container is considered ready. 

- But if I had run kubectl get pods a little more quickly. We might have actually seen a zero out of one during that initial five second delay before the readiness probe run for the first time. 

- So that's what this ready column means is it reflects which containers within the pod are actually considered ready to receive requests. 

