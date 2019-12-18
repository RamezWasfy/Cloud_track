# Creating Pods

* Pods are one of the most essential Kubernetes object types. Most of the orchestration features of Kubernetes are centered around the management of Pods. In this lesson, we will discuss what Pods are and demonstrate how to create a pod. We will also talk about how to edit and delete pods after they are created. The principles discussed in this lesson for managing pods apply to the management of other types of Kubernetes objects as well.

- Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/

- Lesson Reference

Create a new yaml file to contain the pod definition.

$ vi my-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    
- Create a pod from the yaml definition file:

$ kubectl create -f my-pod.yml
 
 - Edit a pod by updating the yaml definiton and re-applying it:

$ kubectl apply -f my-pod.yml

- You can also edit a pod like this:

$ kubectl edit pod my-pod

- You can delete a pod like this:

$ kubectl delete pod my-pod


- lesson-Creating Pods
-------------------------
 - A pod is the basic building block of any application running in Kubernetes, Kubernetes is all about managing containers.
 
 - Any containers that are being managed by Kubernetes. You can think of them as running inside a pod, so a pod consists of one or more containers, It's often just one, but they can have more than one. And in addition to one or more containers, a pod also has a set of resources that are shared by those containers. So one or more containers, plus some shared resource, is what make up a pod.
  
 - Now we have a basic pod in Yaml format.
 
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    
  When we're creating objects throughout this course, for the most part, we're going to be working with that Yaml format. 
  
  - This Yaml is all about many Kubernetes objects. They're going to look very similar to this ,I want to make sure that you understand what's going on inside this file. 
  
  - So first, I'm going to specify an API version And every yaml definition for kubernetes object has an API version. So this just specifies what version of the Kubernetes API this data is compatible with. I'm going to do the API version.
  
  - Second thing is that I'm going to do a kind. And if I'm creating a Kubernetes object, I always need to specify that kind. This is the object type , There are many different object types in Kubernetes. So we need to tell the cluster what type of object we want to create. 
  
  - We're going to create a pod, so I'm just going to put a kind of pod now. Objects in Kubernetes also have metadata, and so I'm going to add a metadata field here, and things like the name of the pod are considered metadata. So I'm going to enter the name and I'll just call it my-pod.
 
 - Inside metadata, you can also have labels, Most objects have labels. They can be used to keep your objects organized as well as to apply different types of automation within the Kubernetes cluster, two groups of pods. So it's a good idea to give your pod a label that specifies what application it's associated with. And I'm just going to call the app my app.
  
 - So I'll put labels and then under labels, I have a label called app, and the value of that label is myapp. So now I'm ready to go ahead and provide the specifications. 
 
 - All Kubernetes objects have a specification and a status. Now, we don't need to provide the status in our definition here, we only need to provide the specification. So I'm going to put spec, and what I'm going to do is I'm going to put containers. 
 
 * Note: A pod can run one or more containers. 
 
 - So here is where I specify what containers my pod is going to run. Now, this pod is just going to be one container. So I'm just going to list a container here and I will give it a name myapp-container. Now I also need to specify a container image. This is the image that's actually going to be run for this container. So I'm just going to specify the BusyBox image. 
 
 * Note: 
   Busybox image is just a small, lightweight image that doesn't do a whole lot and it's used a lot for testing, so since we're just doing some kind of testing and creating a pod, that's a great image for us to use for our purposes right now. And then I'm going to specify a command now.
  
  * Note: 
    Not every image needs a command. Some images have a built in command, and if I leave command blank then those images will run just fine, they'll just run their built-in command. 
  
 - However BusyBox image requires a command. So we need to specify just a simple command that we're going to run inside this container. And I'm just going to do a shell command. And this does take an array. That's just the data type that is expected here for the command, and I'm going to run echo Hello Kubernetes! So basically what this container is going to do is it's going to print this inside the container and then it's just going to pause for 10 minutes and then it will stop and the pod will shut down.
  
 - So we're really not doing a whole lot. We're just running a simple command inside this BusyBox container. So I'm going to go ahead and save that file and quit.
   
 - Now what we're going to do is we're going to use this Yaml definition to go ahead and create a pod. 
 
 $ kubectl create -f my-pod.yml. 
   
 - So remember, in the last lesson, we talked about all the different types of Kubernetes objects there are. You can pretty much create all those different objects the same way we just created this pod. You create a Yaml definition for the object, of course, different types of objects take different types of fields here and things like that. But you create a yaml definition for the object, and then you can create it using kubectl create -f , and then just pass in that yml.
    
 - Now the pod is created. 
 
 $ kubectl get pods >> you'll notice that it's running, if I were to wait 10 minutes, it would stop because that command would complete and the pod would shut down. But right now it's still running and it's one out of one ready. That means one out of one containers when we only had one container are currently ready and fully up and running. So it looks like everything is working just the way we expect it to. 
 
 - Now, what I'm going to do next is I'm going to show you how to edit this pod. And one of the ways we can edit the pod is just by editing this Yaml definition, I'm going to go in here now. You can't change everything on a running pod. So what I'm going to do is I'm just going to change the label, that's something that's very easy to change. We'll just change it to myapp-1. And then what I can do is do kubectl apply. 
 
 - Now I can't do kubectl create, because if I create it, it's going to say there's already an object with same name and it's going to fail. But if I do a kubectl apply, I can update the existing objects. I'm just going to do -f in passing that edited file and it complained a little bit because if you use kubectl create initially and then do apply, it will complain the first time, but it still updated the pod and everything's fine.
  
 - Another way that you can edit objects is to just use kubectl edit pod, and I need to pass in the name of the pod now. And if you remember, the name is simply my-pod. 
 $ kubectl edit pod my-pod >>
 
 - So on the exam they have an object that's already running in the cluster, and they want you to make a change to some of the properties. Kubectl edit is a very fast and efficient way to do that. As you can see, it just opened my text editor and I can edit the information about the pod right here. Now there's a lot of other stuff that was added on by the cluster, a lot more than we had in our original pod spec. But I could actually come in here and change some of these values and it will successfully update the pot. So I'm going to come in here and change this label to myapp 2, and then if I just save that it immediately edits the pod to reflect that new value again. Not everything can be edited that way. 
 
 - Some things I need to delete the pod and then recreate it in order to change certain values. 
  $ kubectl delete object_type. So if this was some other type of object, I could put node or service or some other object type there. But I'm going to put 
  $ kubectl delete pod my-pod >> And it might take a few seconds because it has to spin down the pod and complete everything. But it will go ahead and delete that pod. 