# SecurityContexts

* Occasionally, it's necessary to customize how containers interact with the underlying security mechanisms present on the operating systems of Kubernetes nodes. The securityContext attribute in a pod specification allows for making these customizations. In this lesson, we will briefly discuss what the securityContext is, and demonstrate how to use it to implement some common functionality.

- Relevant Documentation
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

- Lesson Reference

- First, create some users, groups, and files on both worker nodes which we can use for testing.

$ sudo useradd -u 2000 container-user-0
$ sudo groupadd -g 3000 container-group-0
$ sudo useradd -u 2001 container-user-1
$ sudo groupadd -g 3001 container-group-1
$ sudo mkdir -p /etc/message/
$ echo "Hello, World!" | sudo tee -a /etc/message/message.txt
$ sudo chown 2000:3000 /etc/message/message.txt
$ sudo chmod 640 /etc/message/message.txt


- On the controller, create a pod to read the message.txt file and print the message to the log.

$ vi my-securitycontext-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message


- Check the pod's log to see the message from the file:

$ kubectl logs my-securitycontext-pod

- Delete the pod and re-create it, this time with a securityContext set to use a user and group that do not have access to the file.

$ kubectl delete pod my-securitycontext-pod --now

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  securityContext:
    runAsUser: 2001
    fsGroup: 3001
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message


- Check the log again. You should see a "permission denied" message.

$ kubectl logs my-securitycontext-pod

- Delete the pod and re-create it again, this time with a user and group that are able to access the file.

$ kubectl delete pod my-securitycontext-pod --now

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  securityContext:
    runAsUser: 2000
    fsGroup: 3000
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message

- Check the log once more. You should see the message from the file.

$ kubectl logs my-securitycontext-pod


- lesson-SecurityContexts
-------------------------

- A security context is something we can use to define privilege and access control settings for pods and containers.
- If a pod or container needs to interact with the security mechanisms of the underlying operating system in a customized way, then security contexts are how we can go about accomplishing that
- we can define a security context as part of our pod specifications.

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  securityContext:
    runAsUser: 2000
    fsGroup: 3000
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]

- In this example it is very simple. It just allows us to run our containers as a particular user on the underlying operating system and with a particular file system group defined by the IDs of the user and the file system group, respectively.

- So I'm gonna go over here to my two worker nodes. I'm logged into both of my worker notes. I'm gonna do a series of steps on both of these servers, and what I'm gonna do is I'm just gonna create some users and groups and files that we can use for testing.

- I'm running this set of commands on both of my worker nodes. And what I'm doing here is I'm creating a new user, and I'm giving it a specific user ID just so that I know exactly what the user ID is. I'm giving it a user ID of 2000 and I'm just gonna call this one container-user-0. So gonna create that user on both of my worker nodes.

$ sudo useradd -u 2000 container-user-0

- I'm also going to create a group, and this one is gonna have an ID of 3000 and we'll just call it container-group-0

$ sudo groupadd -g 3000 container-group-0

- I'm gonna go ahead and create an additional user 2001 and I'll also create another group 3001 Container Group one.

$ sudo useradd -u 2001 container-user-1
$ sudo groupadd -g 3001 container-group-1

- Now we have two users and two groups that now exist on both of our worker nodes.

- I'm gonna make a directory so that I can create a file here. And let's just make this directory, /etc/message.

$ sudo mkdir -p /etc/message/

- Now I'm gonna go ahead and create a file in that directory, and I will just echo a message here. "Hello, World!". And because my cloud user doesn't have access to the file, I'm just gonna pipe that to sudo tee and we will put it in that etc/message directory that we just created and we'll call it message.txt.

$ echo "Hello, World!" | sudo tee -a /etc/message/message.txt

- We can use that for some testing to see what are containers are and are not able to access once they're up and running.

- Now let's go ahead and set some permissions on the file. I will do chown here and we'll go ahead and make this file owned by our user and our group that we created earlier So user 2000 and Group 3000

$ sudo chown 2000:3000 /etc/message/message.txt

- So we'll change the ownership on that file. And I'm also going to change the permissions and I'll just change them to 640 So, essentially, if you're not part of this user or this group, you won't even be able to read the file. So in order to read that file, you're going to have to either be user 2000 or be a member of group 3000.

$ sudo chmod 640 /etc/message/message.txt

* Note:
  We've done all those steps on both worker nodes.

- So I'm just gonna come back here where I'm logged into my controller and let's go ahead and create some pods just so we can test how all this works.

- So I'll just create a pod, and I'll call it my-securitycontext-pod. We're just gonna use this pod to test how we can interact with those files. The users and the groups that we just created on our worker nodes. Well, go ahead and just create that Yaml file. And I'm just gonna paste in the pod specifications here.

$ vi my-securitycontext-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message


- Essentially, what we're doing is we're just spinning up a single container, and we're going to read /message/message.txt
- What we're doing is we're actually mounting that file from the file system. we haven't talked about volume mounts yet. We used them a little bit with config maps. But all you really need to know for now is we're going to be actually accessing that file that we just created that message.txt that says hello, world.

- We're gonna be accessing that file by mounting it to our container as a volume mount, and then the container will be able to output the contents of that file to the container logs, so we can basically verify whether or not our container is able to access that file on the actual worker node.

- So this is a very simple pod. We haven't really added any custom security context settings yet, so let's go ahead and just see what happens if we run this pod as is.

$ kubectl apply -f my-securitycontext-pod.yml >> pod is created

$ kubectl get pods >> verify that the pod is up and running

$ kubectl logs my-securitycontext-pod >> to see our container log. here is our message. So we were able to successfully read that file and print out the message.

- Now, we might ask a question, why it works. because we did restrict permissions on that file. So how is our pod able to read that file?

$ kubectl exec my-securitycontext-pod -- ps >> execute ps inside the pod.

- You'll notice our commands here are running as root. So the reason why we're able to access this file on the host is because we're running his root and that is actually mapping back to root on the host as well.

- And you can imagine, that could have some security consequences if somehow the main process of this pod was compromised. If someone was able to gain control of it, they might be able to affect things on the host. So it might be a good idea to allow our pod to run as a more restricted user or file system group. And we can accomplish that using a security context.

- Now we will delete this pod so that we can recreate it.

$ kubectl delete pod my-securitycontext-pod --now >> pod deleted

* Note:
 --now,will actually delete a little bit faster.

- Now iam going to set this pod up with a security context to run as a user and group that do not have permission to access our files. So we're just going to demonstrate the ability to restrict permissions. Using a security context later on will change that so that we actually can access the file with a more limited user.

- So what I'm going to do first is I'm going to add this security context key to my pod spec, and I will add runAsUser to instruct kubernetes which user on the host we want to use when we run this particular pod. And I'm going to set that to 2001. And you may remember this is that second user we created, this user does not have access to see our message file.

apiVersion: v1
kind: Pod
metadata:
  name: my-securitycontext-pod
spec:
  securityContext:
    runAsUser: 2001
    fsGroup: 3001
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "cat /message/message.txt && sleep 3600"]
    volumeMounts:
    - name: message-volume
      mountPath: /message
  volumes:
  - name: message-volume
    hostPath:
      path: /etc/message

- I'm going to use fsGroup and I'll set that to the ID 3001 which is that group that we created on our worker notes that again does not have access to read our message.txt. So gonna go ahead and just save and quit that And let's create our pod again.


$ kubectl apply -f my-securitycontext-pod.yml >> Our pod is created.

$ kubectl get pods >> It looks like there's an error, and that's probably because we're not able to read from that file.

$ kubectl logs my-securitycontext-pod >> Error, we cannot open message.txt Permission denied. So we have successfully restricted what this pod is and is not able to access simply by changing it to a user and group other than root so that it can't necessarily access all files on our host.

- So let's go ahead and fix that

$ kubectl delete pod my-securitycontext-pod --now >> delete the pod so that we can re create it again.

vi my-securitycontext-pod.yml >> Now let's change it to User 2000 and Group 3000. This user in group does have permission to access the file, but doesn't really have permission to do anything else on our worker nodes.

- So this is a restricted user in group that should be appropriate for the security that we want to enforce in this situation.

$ kubectl apply -f my-securitycontext-pod.yml >> apply the new fix

$ kubectl get pods >> This time we're up

$ kubectl logs my-securitycontext-pod >> check the log, there's our hello world message coming from message.txt.

- I've shown you how to run as a specific user and with a specific file system group.

- Now, if you check the documentation on security context, you can find a lot of different settings that you can use.
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
