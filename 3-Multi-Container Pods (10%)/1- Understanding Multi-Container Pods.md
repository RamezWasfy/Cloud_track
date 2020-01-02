# Understanding Multi-Container Pods

* Multi-container pods provide an opportunity to enhance containers with helper containers that provide additional functionality. This lesson covers the basics of what multi-container pods are and how they are created. It also discusses the primary ways that containers can interact with each other within the same pod, as well as the three main multi-container pod design patterns: sidecar, ambassador, and adapter.

* Be sure to check out the hands-on labs for this course (including the practice exam) to get some hands-on experience with implementing multi-container pods.

- Relevant Documentation
https://kubernetes.io/docs/concepts/cluster-administration/logging/#using-a-sidecar-container-with-the-logging-agent
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

- Lesson Reference

- Here is the YAML used to create a simple multi-container pod in the video:

apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.15.8
    ports:
    - containerPort: 80
  - name: busybox-sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do sleep 30; done;']

- lesson-Understanding Multi-Container Pods
--------------------------------------------

- Multi container pods: They are simply pods that have more than one container, and usually these containers are working together and sort of forming a single unit. 

- It's not a good practice Toe have multiple unrelated containers, all contained within the same pod, so you have a multi container pod. The multiple containers are working together and connected to one another in some way. 

- So let's go ahead and just create a basic multi container pod in the cluster. 

`vim multi-container-pod.yml`

- This is Just a basic pod definition. I'm calling it multi-container-pod and I have a container running nginx and I've got a container port. So when nginx is listening on Port 80 I'm exposing that port to the cluster.
 
- Now, to make this a multi container pod, all I really need to do is just add another container here under the list of containers, so I'm going to call this one busybox-sidecar, and we'll just run the busybox image. And I just have a command here that is going to run sleep 30 in a loop. So this container really isn't doing anything. It's just going to sleep.
 
- This isn't a setup that you would use all that often in the real world, because these containers really aren't directly interacting with each other in any real way. So the busy box sidecar really isn't doing anything. It's just running in a loop, but technically this is a multi container pod. 

`Kubectl apply -f multi-container-pod.yml` pod is created it says, ready out of two

- So that means there's actually two containers in this pod, So we can see that both of our containers are ready. So everything's up and running with our multi container pod. 

- Now let's dive into some of the real world examples and some of the design patterns that we can use to actually utilize multi container pods in different ways. 

- let us have a look at this little interactive diagram called Multi Container Pods, this can show you how multi container pods work. First thing I want to talk about is how containers can interact with one another. 

- There are three primary ways that containers can communicate or interact with one another when they're sharing the same kubernetes pod. 

1- First way: They can interact through the shared network, so containers running in the same pod share the same network space. From a networking perspective, it's as if the two containers were running on the same host. In fact, they can access each other simply using localhost. So if container to is listening on port 1231234 then container-1 can always communicate with container-2, simply by accessing localhost 12324 And that holds true even if that port is not exposed to the cluster or anywhere outside of the pod itself. Containers can communicate with each other on all ports within the context of the same pod. 

- This is one way that containers can interact is simply by communicating using that shared network.
 
2- Second way: is to use a shared storage volume so we can take a storage volume which contains some files on the disk. And we can actually mount that same volume to two different containers so that they can both interact with the same files so you could have one container outputting files to the volume and the other container reading them, or you could have both of them reading and writing files to that same volume. 

- There's a lot of different ways that you can have containers in the same pod interact with each other using a shared storage volume.
 
3- Third way: is through shared process namespace, and essentially what this does is it allows the two containers to signal one another's processes. In order to implement that, you have to add an attribute to your pod spec called shareProcessNamespace and set that too true. But once that is set to true, your containers can actually interact directly with one another's processes, using a shared process namespace. 



- Now let's look at some common multi container pod design patterns, this design patterns can give you a little bit more of a concrete idea of how multi container pods can really be used.
 
- We have 3 different design patterns,  sidecar pod, ambassador pod and adapter pod design patterns, and these. There are three different ways of utilizing multi container pods, 

1- Sidecar: We've got our pod. We've got a main container and a sidecar container. The sidecar design pattern is fairly general doesn't really express how these pods are communicating with one another. But essentially, what occurs here is the sidecar container enhances the functionality of the main container in some way. So one example of this could be, say, a sidecar container that syncs files from a git repositories to the file system of a Web server container. So essentially, you could use that for automated deployments. You could have a Web server serving static files and a sidecar that say every two minutes checks for new versions of those files. If the files have been updated, it pulls in the new files and pushes them into the file system of the main container, so they're automatically updated without even having to restart or redeploy that container. 

- So a sidecar is just something that kind of sits alongside the main container and run some kind of automated process or does something that enhances the value of that main container.
 
2- Ambassador pods, and these are a little bit more specific, Ambassador pods are all about capturing and translating network traffic. So you can see in the diagram we have network traffic coming in and it does not go directly to the main container. It goes to the ambassador first and then to the main container. So this ambassador container could be doing a variety of things to that traffic. It could be transforming it in some way or capturing data about it in some way. One example of this would be an ambassador container that listens on a custom port and forwards traffic to the main container on a hardcoded port. 

- So if you have a container that is hardcoded to only, listen on a specific port. So you've got some legacy piece of software, and the port is not easily configurable. But you wanted to listen on a different port. You could use an ambassador container too easily. Solve that problem simply by forwarding the traffic from one port to another. And in fact, the lab at the end of this section of the course is actually all about that. 

3- Adapter pod: adapter pods are all about changing the output from the main container. So whatever that output is, whether it's information going into a file or simply just consul output, adapter containers transform and change that output in some way in order to adapt it to some other type of system. So we have output coming from the main container, and the adapter container is formatting or changing that output and then making that output available outside the pod itself. So one of the common use cases for this is if you have some kind of a log analyzer, something like Prometheus or logstash or something like that. You can use an adapter container to apply specific formatting to those logs and allow them to be pushed or pulled to some kind of external system. 

- So adapter containers conserve a wide variety of use cases, but they're all about transforming the output from the main container. 

- Those are the three main use cases for multi container pods. If you want to learn more about that. I've got some links down below this video to some documentation, and you can also check out the labs in this course because the labs like the one at the end of this section as well as the practice exam. For this course, we'll show you how to implement some of these design patterns when you're building multi container pods.
