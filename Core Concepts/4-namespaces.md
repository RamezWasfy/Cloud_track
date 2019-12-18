# Namespaces

* Although namespaces are not on the objectives list for the CKAD exam, they play an important role in many of the tasks that the exam may cover. An understanding of namespaces is necessary in order to avoid confusion in many scenarios that may arise when working with Kubernetes. In this lesson, we will briefly discuss namespaces, how to assign objects to namespaces, and how to browse objects within namespaces.


- Relevant Documentation
https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

- Lesson Reference

- You can get a list of the namespaces in the cluster like this:

$ kubectl get namespaces

- You can also create your own namespaces.

$ kubectl create ns my-ns

- To assign an object to a custom namespace, simply specify its metadata.namespace attribute.

apiVersion: v1
kind: Pod
metadata:
  name: my-ns-pod
  namespace: my-ns
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    
- Create the pod with the created yaml file.

$ kubectl create -f my-ns.yml

- Use the -n flag to specify a namespace when using commands like kubectl get.

$ kubectl get pods -n my-ns

- You can also use -n to specify a namespace when using kubectl describe.

$ kubectl describe pod my-ns-pod -n my-ns


- lesson-Namespaces
-----------------------

- Name spaces are essentially a way to keep objects categorized and organized within your kubernetes cluster. Most kubernetes objects are assigned to a name space, and if you create a pod or other type of object and you don't specify the name space, that object will automatically be assigned to the default namespace. 

- So any time you don't specify in name space in Kubernetes, you're automatically working with the default namespace. 

- Namespaces can be used for a variety of purposes. One example would be if you have multiple different applications running in the same cluster. You may take all the different pods and other objects that are associated with each one of those applications and make sure that each application has its objects inside of its own. 

- Namespaces just to kind of keep things organized. Another way to use namespaces would be if you have multiple teams in a company all working with the same kubernetes cluster. You may give each of those teams access to their own name space so that they can work with the cluster without interfering with what the other team is doing. 

- So let's just take a look at the name spaces that currently exist in our cluster.
$ $ kubectl get namespaces >> As you can see, these are the default name spaces, the ones that come with the cluster When we set it up, 

- We have the default namespace, kube-public and kube-system. We're going to go ahead and create a custom name space, and then we will create a pod that is assigned to that namespace. 

$ kubectl create ns my-ns >> to create a namespace, ns is just the abbreviation for namespace. And I'm just gonna call it my-ns. 

- And now let's go ahead and create a pod that is in this name space. I'm just gonna create a Yaml file and I'm going to call it my-ns-pod.yml  

apiVersion: v1
kind: Pod
metadata:
  name: my-ns-pod
  namespace: my-ns
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
 
- I've named the pod my-ns-pod. And this is how I can specify the namespaces right here under the metadata. So metadata namespace, and then I list the namespace I want, which in this case, is just gonna be that my-ns namespace that we created a moment ago. 

-  Save that file, and then I will create that pod. 

$ kubectl create -f my-ns-pod.yml >> So now I've created the pod.

- At this point, I'm going to do a kubectl get pods and you'll notice it says no resource is found. The pod that I just created, it doesn't even show.

$ kubectl describe pod pod my-ns-pod >> It says that the object is not found and that can be very confusing when you're working with kubernetes. 

- Well, one of the causes of that confusion could be that the object is in a different namespaces. So when we just do a kubectl get pods we are actually only searching the default namespace. It's exactly the same as if we had specified that namespace like this. And that -n flag is how we specify the namespace that we want to work with. So if I don't specify the namespace, I'm looking in default automatically. And what I want to do is I want to instead look in my-ns. And when I look in the proper namespace, we can see the pod actually shows up, and I could do the same thing with kubectl describe my-ns-pod, and then I need to put the -n and specify the namespace there. 

- And sure enough, now it's able to appropriately describe that pod. So whenever you're working with kubernetes objects that are not in the default namespace, make sure that you always add that -n flag and specify which namespace you want to work with. Otherwise, the cluster is going to act like it can't find the object or like it doesn't even exist
