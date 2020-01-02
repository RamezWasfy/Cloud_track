# Basic Container Configuration

* When building applications in Kubernetes, it is often necessary to provide some configuration for your containers. In this lesson we will discuss some frequently-used container configuration options: command, args, and containerPort. After completing this lesson, you will have a basic understanding of some the ways in which Kubernetes allows you to customize how your containers are run within a pod.

- Relevant Documentation
https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/

- Lesson Reference

- You can specify custom commands for your containers.

apiVersion: v1
kind: Pod
metadata:
  name: my-command-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo']
  restartPolicy: Never


- You can also add custom arguments like so:

apiVersion: v1
kind: Pod
metadata:
  name: my-args-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo']
    args: ['This is my custom argument']
  restartPolicy: Never
  
  
- Here is a pod with a containerPort:

apiVersion: v1
kind: Pod
metadata:
  name: my-containerport-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx
    ports:
    - containerPort: 80


- You can check the status of your pods at any time >>  kubectl get pods 

- lesson-Basic Container Configuration
---------------------------------------

- In this lesson will be talking about some of the basic configurations that we can do in order to customize how we run our containers within pods. 

1- commands: When you run a container in a pod, you have the option to specify a custom command. 

* Note:  
  Some container images come with a default command already specified. And if that's the case, then many times we don't need to specify custom command. But occasionally we need a little bit of additional control. We need to be able to customize the command that gets run inside the container. And this command option is what allows us to do that.
 
- Now let's begin by creating a pod with a custom command. So I'm going to go ahead and create a Yaml definition file, and I'm just gonna call it my-command-pod.yml
 
apiVersion: v1
kind: Pod
metadata:
  name: my-command-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo']
  restartPolicy: Never
  
- I'm calling the pod my-command-pod and what I'm doing is I'm passing in this command option to the busy box container image. And this command option is specifying that we're going to just run the command 'echo' . Now that command is going to print something to the screen. Or in this case, it's gonna print nothing to the screen because we haven't specified anything to actually print. 

- Now the container is going to stop because the command is already completed. So I'm gonna add in this restart policy never. Otherwise kubernetes is gonna continue trying to restart that container because it completed and shut down.  

- Therefore we have set restartpolicy to never so we don't accidentally continually try to restart this container. So all I'm doing is I'm specifying this custom command, and that's the command that's gonna run inside this container when it fires up.
 
- Now we create that pod. And if I do kubectl get pods, you can see it's already in a status of completed, that means that the command completed the container has shut down and Kubernetes is not going to try to restart that container unless I tell it to. 

2- args: custom arguments. So while the command option lets you customize the command that runs inside the container, the args option lets you specify the arguments that come after the command. 

- Let's go ahead and just create another pod called my-args-pod

apiVersion: v1
kind: Pod
metadata:
  name: my-args-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['echo']
    args: ['This is my custom argument']
  restartPolicy: Never
  

- We are specifying a command of Echo. And then I'm specifying args So the one that we create earlier would actually echo nothing because we didn't specify any argument following the Echo Command. But this time we're going to specify this string as an argument that's going to follow the command that gets run. So it's gonna echo this text to the consul. 

- And when that's finished, the container is gonna go ahead and shut down. and then I'll create that pod , as you can see It's already finished running, so it's in the completed status. 
 
3- containerPort: this is a pretty important option many times when we're creating pods. We need things outside of those pods to be able to communicate with them via the Kubernetes Cluster Network. Whether that is other pods communicating with this pod or potentially even service's and entities that are outside the kubernetes cluster communicating with our pods and ultimately with our containers. 

- In order to do that, we need to expose container ports using the container port option. If we don't expose the container ports, you may have a container running in a pod and listening on a port. But that port will only be accessible from either that same container or other containers running within the same pod, in order to make that port accessible to anything else within the kubernetes cluster or anything else in the outside world. We need to expose that port using the container port option. 

- So I'm going to go ahead and just create a pod called my-containerport-pod. 

apiVersion: v1
kind: Pod
metadata:
  name: my-containerport-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx
    ports:
    - containerPort: 80
    
    
Now this one is not using the busy box image because it doesn't really make sense to create a container port. If we don't have something actually listening on the port. So I'm just gonna create a simple nginx server, and by default, nginx is gonna listen on Port 80 because it is just a regular old Web server. 

- So if we want anything else in the cluster to be able to access that nginx server, we need to specify a list of ports and put a container port of Port 80 and that's gonna allow other things in the cluster to be able to talk to this nginx container on Port 80. 

- I'll create the pod. Now this one is not going to automatically complete because it's going to continue to run that nginx server. So as you can see, it's in the container, creating status right now going to run that again. And now we see it is fully up and running. 
