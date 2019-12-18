# ConfigMaps

* Management of configuration data is one of the challenges involved in building and maintaining complex application infrastructures. Luckily, Kubernetes offers functionality that helps to maintain application configurations in the form of ConfigMaps. In this lesson, we will discuss what ConfigMaps are, how to create them, some  of the ways that ConfigMap data can be passed in to containers running within Kubernetes Pods.

- Relevant Documentation
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

- Lesson Reference

Here's an example of of a yaml descriptor for a ConfigMap containing some data:

apiVersion: v1
kind: ConfigMap
metadata:
   name: my-config-map
data:
   myKey: myValue
   anotherKey: anotherValue



- Passing ConfigMap data to a container as an environment variable looks like this:

apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: my-config-map
          key: myKey
          
          
- It's also possible to pass ConfigMap data to containers, in the form of file using a mounted volume, like so:

apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-volume-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(cat /etc/config/myKey) && sleep 3600"]
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config-map
        
        
- In the lesson, we'll also use the following commands to explore how the ConfigMap data interacts with pods and containers:

$ kubectl logs my-configmap-pod

$ kubectl logs my-configmap-volume-pod

$ kubectl exec my-configmap-volume-pod -- ls /etc/config

$ kubectl exec my-configmap-volume-pod -- cat /etc/config/myKey


- lesson-ConfigMaps
-------------------------

- Configmap: A type of kubernetes object that represents a key value store of configuration data. So if you have configurations that you need to pass into your pods and your containers, it's often a good idea to store those configurations in a CONFIG map. They offer you a centralized place to store your configuration data within the kubernetes cluster. And once you've created that key value data inside of your config map, you can pass it into your containers so that they can use it. And then if you need to update or change those configurations, it's just a matter of updating the config map and potentially restarting any pods that are using that data. 

- Now we're gonna go ahead and create CONFIG map then apply it using kubectl create or kubectl apply to create that object in the cluster. 

apiVersion: v1
kind: ConfigMap
metadata:
   name: my-config-map
data:
   myKey: myValue
   anotherKey: anotherValue


- So I'm gonna go ahead and create a file, and I'm just gonna call it my-config-map.yml ,we need to provide an API version. So I'll just put v1 and then the kind which specifies the type of object we want to create. So I'll just create a ConfigMap and just like we would do with pods if we want to give our config map a name, we need to specify metadata name and I'll just call it my-config-map.
 
* Note : 
  Now this metadata name is important because if we want to pass our data into some pods, we're gonna need to know that name in order to reference our ConfigMap. 

- Now I need to go ahead and specify the actual data that this CONFIG map is gonna contain. So I'm just gonna add a data key here. And underneath this data key, I can add really any type of key value data that I want. So I'm just going to create one called myKey with myValue. So there's some data and I can add as many keys as I want under here, so I'll just go ahead and add another one called anotherKey with the value of anotherValue So that's my config map.
 
 $ kubectl apply -f my-config-map.yml >>  the CONFIG map is created within the cluster. 
 
- So now I want to go about passing our config map data into an actual pod so that our containers can use it. And there's multiple different ways to do this. 
 
1- The first way is that we can actually mount our config map data as an environment variable that is exposed and accessible to our containers. So when your container application is running, it will be able to see that data contained within that environment variable. 

- Passing ConfigMap data to a container as an environment variable looks like this:

apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: my-config-map
          key: myKey
          
- So I'm gonna go ahead and create a pod to show you how to do that. And I'll just call this one my-configmap-pod.yml and of course I need my API version. The kind is pod and I'll specify the pod name of my-configmap-pod. 

- Then I'm gonna create my pod specification. This is just a simple, busybox container that's just going to echo this environment variable MY_VAR, and then it's going to sleep for 10 minutes. 

- So I want to show you how to go ahead and pass that configuration data in as an environment variable, and we're actually gonna pass it in as a variable called MY_VAR. So this line right here is actually going to echo the data that came from our CONFIG map to the console within the container. So the way that we can specify environment variables in order to pass them into containers is i can specify this env key. That's part of the container spec. 

- And there's a lot of different ways to specify environment variables, and you would do them all using this env key. But what we're gonna do is we're gonna pull in our config map data and turn that into an environment variable. So I'm going to specify one here, and I'm going to give it a name of MY_VAR. And the name is gonna be the name of the environment variable within the container, and that's what we're going to use to reference it. So that's why I'm referencing MY_VAR here that matches this name and therefore this reference is going to point to this environment Variable. Now, the next thing I want to specify here about my environment variable is I'm going to use this value from key, which is telling the cluster that we're going to pull this value in from somewhere else. So we need to specify that we want to pull the value in from our config map. So I'm going to use a config map key, ref, which is going to reference a specific key inside of a specific CONFIG map. So I'm going to pass in the name you may remember we called our config map my-config-map. So that's the name of the config map that we created. And then I'm going to reference the actual key. And you may remember that the first key that we put into our config map was just called myKey, so it's going to go into this CONFIG map. It's going to look for this key, and the environment Variable will be set to the value of that key. So that's how you can reference config Map data as an environment variable within one of your containers. 

- Save that file and we'll create the pod. >> $ kubectl apply -f my-configmap-pod.yml

- Make sure that the pod has started correctly >> $ kubectl get pods >> So there it is. my-configmap-pod is up and running. 

- Now let's go ahead and look at the container log from my-configmap-pod >> kubectl logs my-configmap-pod 

- Based on the command from our container, we should see the value of our environment variable, which is coming from the config map. That value should be printed to the log. You can see the value of mykey from the config map, which we transformed into that environment. Variable being printed to the container log right there. 



2- Another way that we can pass in config map data to one of our containers. We don't have to do it as an environment variable. We can also do it as a mounted volume. In other words, a file that is mounted to our container that the container will then be able to access. 

apiVersion: v1
kind: Pod
metadata:
  name: my-configmap-volume-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(cat /etc/config/myKey) && sleep 3600"]
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: my-config-map
        
        
- In order to demonstrate that I'm just going to create a new pod and I'll just call it my-configmap-volume-pod , I'll just paste in some basic configuration here. We're just creating a pod with a single busybox container. Now what we're gonna do is we're going to mount our configuration data from the config map as a mounted volume.
 
- Now, we haven't talked about mounted volumes yet. I'm just gonna show you how to go ahead and mount it as a configmap. And what we can do is add this volume's key down here to provide a list of volumes that we want to create for this pod. And I'm just gonna give this one a name and because it contains our configuration data from the configmap, I'll just call it config-volume and then I can add this configMap key which is going to instruct kubernetes that we want this volume to represent some data from a configmap. So I'll just give it a name and then reference the name of our configmap, which is just my-config-map. 

- Now that I've created the volume, I also need to go ahead and mount it to the actual container. So I'm gonna add this volumeMounts key here and then I need to provide the name of the volume that I want to mount, which is just this name that we set down here just config-volume. And then I also need to set a mountPath which is going to be the path on the file system where this data is going to be mounted. 

- So I'm just gonna mount to /etc/config because its configuration data, that seems like a good place to go ahead and mount that volume. So in order to properly demonstrate this, we should go ahead and access that data once it is mounted. So I'll just do that with a custom command and we'll just do sh -c and then we will echo that data. So I'm just gonna echo the results of another command. I'm just going to cat /etc/config/myKey .
 
- So when we mount our config map at this location, what's gonna happen is each top level key from our config map is going to become a file underneath this path. So we have a key in our config map called Mikey. Therefore, there's going to be a file under /etc/config/myKey And that file is going to contain the value of the key.
 
- So when we cat this file, it should print the value of my key from the config map to our container log. So also, just to make sure that this pod doesn't shut down immediately going to add a sleep statement here to have it sleep for one hour after it prints this information to the log. So with that, I'm going to go ahead and save my file. 

- kubectl apply >> now it looks like my pod was successfully created. 
- kubectl get pods >> I can see that my pod is up and running, so it looks like everything is working so far. 

- So let's access the logs and we can see that it correctly printed my value to the container log, which was accessed via amounted volume. 

- Now, I also want to give you a little bit of a clearer idea of what's going on here. So what I'm gonna do is I'm gonna use Cube C t l execs to execute a command inside my config, Matt Volume pod. That'll put Dash, Dash. And here we can put the command that we want to execute. So I want to show you what's going on in the file system here with this mounted volume. So I'm just gonna do in ls on /etc/config. That's where we mounted our config map data. 

- As you can see, we have two files here, mykey and anotherkey. So each one of those top level keys that we put into our config map is a separate file right here on the file system. And of course, if I can't at sea slash config slash mikey, I can see that the value there is my value. And if I can't the another key file, I get the value of that key, which is just another value. So essentially, what's going on here when we mount this data to the file system is each one of those top level keys is big becoming a file which inside that file, contains the actual value of that key. So that's just another way to access your CONFIG map data inside one of your containers. Now we've talked about how to create convict maps, and I've also shown you two different ways to access CONFIG maps inside your containers, both using environment variables and mounted volumes. That's all for this lesson. See you in the next one

