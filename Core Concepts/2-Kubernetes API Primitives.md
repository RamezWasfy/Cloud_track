# Kubernetes API Primitives

- Kubernetes API primitive, also known as Kubernetes objects, are the basic building blocks of any application running in Kubernetes. Building and managing Kubernetes applications means working with objects. It is essential to be familiar with Kubernetes objects in order to develop applications for Kubernetes. In this lesson, we will discuss what Kubernetes objects are, and how to find information about objects in a cluster.

- Example of kubernetes Objects: 

1- Pod
2- Node
3- Service 
4- Service Account

$ kubectl api-resources -o name >> give all objects in kubernetes clusters

- Kubernetes API Primitives are data objects that define the state of the cluster. Each object has a spec and a status.
- Spec - defines the desired state. 
- Status - describes the current state.

$ kubectl get >> returns a list of object types available to the cluster.
- Define an object’s spec in the form of yaml data, for use for creating and modify objects.

$ kubectl describe $object_type $object_name  >> Obtain the spec and status

$ kubectl get $object_type $object_name >> You also get information about an object

Note : 
* Add the -o yaml flag to get the data in yaml format.

# Hands on 

$ kubectl get pods >>  it's going to return a list of pods now, however We haven't created any pods in our cluster yet, at least not any manually created pods. 

- So if I just do kubectl get pods, you can see that it says there are no resource is found. However, there are some background system pods.
- If I repeat that command and I specify a namespace called kube-system .

$ kubectl get pods -n kube-system >> You will see now that there is actually a list of several pods running in that namespace and these are all background pods that kind of doing some of the background work of the Kubernetes cluster.

$ kubectl get nodes >>  gave us a list of the nodes in our cluster. So basically, what I'm doing is I'm saying kubectl And then I'm specifying the object type pods or nodes or service is or whatever type of object I want to get. And that's returning a list of the objects of that type that are currently in the cluster. 

- Another thing I can do is I can get more information about an individual objects. 

$ kubectl get nodes k8s-worker1 >> this time, it only returned that one node. Now what I'm gonna do is I'm gonna run that command again. This time I'm gonna put -o yaml and that's going to return the data about this note in a Yaml format. I'm actually gonna get a lot more data if I run that command. There's a huge amount of data here and what this is this is a yaml representation of this object. This node that's currently within the cluster has all the information here. We can see the metadata. We can see the name of the node here, and oh, look, there's a specifications when the node was created. There's really not a lot of data here for the specifications, but that's the specifications for this node, and right below that we have the status, and there's quite a lot of data here. This data is populated by the kubernetes cluster itself. It's just all the information about the current state of that node. So that object, just like any other object in the cluster, has a spec and a status. So things to keep in mind when you're taking the exam is that you want to use kubectl get and then the object type to get a list of objects. 

- I'm going to get a list of nodes again. And there's one other command I want to show you.

$kubectl describe >>  So I'm gonna do kubectl describe this first node, so I'm gonna put Node and then I'll just copy that node name. And what kubectl describe does is it kind of gives you a condensed and somewhat more readable overview of the specifications and status of this individual objects. So it doesn't really have the yaml format where we can actually see the spec and status, But all of the information, it's kind of contained right here because this is a node. We can see information about memory usage and resource usage on that node information about the operating system, just all kinds of information about that note and its specifications and status. So this is gonna be very useful for troubleshooting on the exam if you're asked to find information about particular kubernetes object that kubernetes describe command is a great way to do that.






