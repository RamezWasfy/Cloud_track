# Resource Requirements

* Kubernetes is a powerful tool for managing and utilizing available resources to run containers. Resource requests and limits provide a great deal of control over how resources will be allocated. In this lesson, we will talk about what resource requests and limits do, and also demonstrate how to set resource requests and limits for a container.

- Relevant Documentation
https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container

- Lesson Reference

- Specify resource requests and resource limits in the container spec like this:

apiVersion: v1
kind: Pod
metadata:
  name: my-resource-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"


- lesson-Resource Requirements
------------------------------

- Managing resource requirements for our kubernetes applications. 

- Kubernetes gives us the ability to specify the resource requirements of each container when we're creating our pod specifications.
- So containers, memory and CPU requirements can be defined in terms of resource requests and limits. 

- Resource requests: They are something that allows you to specify the amount of resources that are necessary to run a container, and what they do is they govern which worker nodes the containers will actually be scheduled on. 

- So when Kubernetes is getting ready to run a particular pod, it's going to choose a worker node based on the resource requests of that pod's containers. And kubernetes will use those values to ensure that it chooses a node that actually has enough resources available to run that pod. 

- Resource limits: A maximum value of how much resources is the containers in that pod are going to use, If the container goes above that maximum value then it's likely to be killed or restarted by the kubernetes cluster. 

- So resource limits just provide you a way to put some constraints around how much resource is your containers are allowed to use and prevent certain containers from just consuming a whole bunch of resources and running away with all the resources in your cluster, potentially causing issues for other containers and applications as well. 

- So I'm gonna go ahead and create a new pod, and I'll just call it my-resource-pod. I'm just gonna create a yml descriptor for that pod.  

apiVersion: v1
kind: Pod
metadata:
  name: my-resource-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
        
        
- So if I want to add resource requests and limits, I can do that inside the container spec right here by just adding this resources attributes. And then I will add a requests attributes

- Here's where I can list my resource requests So we're going to set resource requests for memory and CPU, so I'll set one for memory, and I'm going to set that to 64Mi that notation there just means 64 mebibytes. So that means we expect this container to use about 64 megabytes. 

- Now, In reality, this little busy box container is probably not going to use even close to that because it's not really doing anything. But in the real world, you'd wanna have kind of an idea of how much memory your application needs and set the resource request to that value.
 
- So let's go ahead and set a CPU request as well, and I'm gonna set that to 250m. Now, the way this notation works is 250m means 250 Millis CPUs, a Millis CPU is 1 1/1000 of a CPU core. 

- So in kubernetes, CPU usage is measured in terms of the percentage of one CPU core that this thing is using. So 250 milli CPU use 251 thousands of a CPU core that's equal to 0.25 or 1/4 of one CPU core. So this means that we expect this application to use about 64 megabytes of memory and 1/4 of one CPU core. 

- So let me go ahead and set my resource limits, and I just set those again right under that resources key. I can add this limits key, and I can add my memory and CPU here. So I'm going to go ahead and put 128Mi for the limit. So we expect it to use 64 megabytes. If, however, it jumps up above 128 mebibytes, this container is going to be killed and potentially restarted, depending on how it's configured. 

- So I'm going to go ahead and add a CPU limit as well, and we'll add that as 500m. So if this container starts using more than half of one CPU core, then that's gonna hit that limit and the cluster is going to behave accordingly. So I'm gonna go ahead and save that file. 

$ kubectl apply -f my-resource-pod.yml >>  The pod was created 
$ kubectly get pods >> The pod is up and running. 

$ kubectl describe pod my-resource-pod >> to see information about the resource requests and limits of a pod. 

- So if you have a pod that's already running in the cluster, and you need to know what the resource requests and limits are, you can see that information right here when you run kubectl describe. So we've got our container, which is called myapp-container. And here's the limits and requests for that particular container.