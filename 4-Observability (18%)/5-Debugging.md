# Debugging

* Problems will occur in any system, and Kubernetes provides some great tools to help locate and fix problems when they occur within a cluster. In this lesson, we will go through the process of debugging an issue in Kubernetes. We will use our knowledge of kubectl get and kubectl describe to locate a broken pod, and then explore various ways of editing Kubernetes objects to fix issues.

- Relevant Documentation
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/
https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/

- Lesson Reference

- I prepared my cluster before the video by creating a broken pod in the nginx-ns namespace:

`kubectl create namespace nginx-ns`

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: nginx-ns
spec:
  containers:
  - name: nginx
    image: nginx:1.158
    
    
- Exploring the cluster to locate the problem

`kubectl get pods`

`kubectl get namespace`

`kubectl get pods --all-namespaces`

`kubectl describe pod nginx -n nginx-ns`

- Fixing the broken image name

`kubectl edit pod nginx -n nginx-ns` >> Edit the pod:

- Change the container image to nginx:1.15.8.

- Exporting a descriptor to edit and re-create the pod.
- Export the pod descriptor and save it to a file:

`kubectl get pod nginx -n nginx-ns -o yaml --export > nginx-pod.yml`

- Add this liveness probe to the container spec:

livenessProbe:
  httpGet:
    path: /
    port: 80
    
- Delete the pod and recreate it using the descriptor file. Be sure to specify the namespace:

`kubectl delete pod nginx -n nginx-ns`

`kubectl apply -f nginx-pod.yml -n nginx-ns`

- lesson- Debugging
--------------------

- Debugging is simply the process of locating and fixing problems within the kubernetes cluster.  

- So I've actually prepared my cluster prior to this video by creating a broken pod somewhere in the cluster and gonna show you the process that I would go through in order to locate that pod and then fix it.
 
# locating the problem: 
 
- If you know that something's broken. you don't always know specifically what pod or what object is broken or where that issue is located in the cluster. 

1- `kubectl get pods` >> it says no resource is found. because I'm looking in my default namespace and there is no pods there. Now we are sure that our problem is not actually in the default namespace. 

`kubectl get namespace` >> you can see we've got several namespaces in the cluster and the problem could be in any one of those namespaces.
 
`kubectl get pods -n SPECIFIC_NS` >>  to look in specific namespaces

`kubectl get pods --all-namespaces` >> to look for pods in all namespaces
 
- As you can see, we have a pod that is currently not in the running status. So this is our pod that's broken. so the broken pod is in the nginx-ns namespace and the pod name is simply is nginx. 

- Now we can see that it's in the ImagePullBackOff status. So there's probably an issue with the image. 

`kubectl describe nginx -n nginx-ns` >> Basically whenever you're doing a kubectl describe and there's a problem with your pod, you definitely need to check out the Events section because there , it will usually give you some kind of idea of what's going on.
 
- So we have a relevant error message here, it says failed to pull image nginx 1.158 

- Now, if you actually look at the nginx image repositories, you'll see that 1.158 is not a valid image tag. There is no 1.158 tag for the nginx image. It simply doesn't exist, and that's why we're failing to pull that image. 

- This particular pod is not able to even start up because it's not able to pull the image. However, sometimes the pod is able to pull the image and start up, and there's a problem actually running the container. 

- In that case, you might need to check out `kubectl logs` to get the logs from inside the container and potentially get even more information about what's going wrong. But in this particular case, we don't need to use kubectl logs because the container isn't even starting up. And so there will be no logs for us to look at.
 
- One of the things you can do in order to fix a variety of different issues is you can use `kubectl edit` 
`kubectl edit pod nginx -ns nginx-ns` >> you can see the broken image name. That's what we need to change if we want to fix this pod. 

- To fix the problem I just need to put a dot there 1.15.8 It looks like I just missed typed the version number, which I did on purpose. 

- So I'm gonna fix that version number and make it 1.15.8 and then I'm just gonna save this file. And when I saved that, that's gonna automatically edit and update the pod.
 
`kubectl get pods -n nginx-ns` nginx pod is up and running, so you've actually located and fix the problem fairly easily. 

# Part 2

- `kubectl edit pod nginx -ns nginx-ns` >> edit the pod but this time i will go to my container spec, and I'm just going to add liveness probe. 

livenessProbe:
  httpGet:
    path: /
    port: 80
 
- Iam just adding a new livenessProbe to this container. save and quit.
 
- You can see that it didn't work. It basically give this error message  `spec: Forbidden pod updates may not change fields other than` and then it's got a list of these fields, so there are many different fields that you cannot change when a pod is already created an up and running, so I'm not going to be able to add my liveness probe to this pod while it is currently running. 

- The only way I can do that is to go ahead and delete and recreate the pod.

- Sometimes when your debugging is in order to make certain types of changes, you're going to need to delete and recreate objects. 

- So how to safely delete a pod and recreate it, because when you delete an object, you lose the object specifications. And if you don't save the object specifications somewhere, then you're not going to be able to recreate that object. 

`Steps:`

1- Save the object specifications >> `kubectl get pods nginx -n nginx-ns -o yaml`

* Note: 
  I'm going to do the output in the yaml format `-o yaml` and also `--export` flag this will exports only the specifications for the object and not the status , without using --export flag i will also get the status and I don't need to preserve the status in order to recreate the object and as you can see, it has created output of that yaml file.
  
- In order to save that to a file re-run that command again and  redirect it to nginx-pod.yaml

`kubectl get pods nginx -n nginx-ns -o yaml --export nginx-pod.yaml` >> now my object specifications is stored in that yml file

- Now i can go ahead and just edit the file to get the changes that I want , so here's my specifications, and I'm going to go down here and add in my liveness probe that I wanted to create.

2- Delete the object then re-create it with the new liveness probe.

`kubectl delete pod nginx -n nginx-ns` >> Pod is successfully deleted
 
3- Recreate the pod 

VIP NOTE:
Now we are going to recreate the pod, `kubectl apply -f nginx-pod.yml` however there is one thing missing which is i need to specify the namespace nginx-ns because when we exported our yaml file, it does not actually include the namespace. 

- So if I were to just go ahead and apply this file as is it would end up putting my pod in the default namespace rather than the namespace that it belongs in, which is in nginx-ns. So if you export an object that's in a custom namespace, make sure that when you go back to recreate it, you specify the namespace so that it goes into the correct name space. 

`kubectl apply -f nginx-pod.yml -n nginx-ns` >> now run this command and check the pod just to make sure that everything is working and it looks like it's up and running.
 
`kubectl describe pod nginx -n nginx-ns`, we should be able to see that Our liveness probe has now been added to this pod. 



