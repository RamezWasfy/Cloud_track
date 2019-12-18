# Secrets

* One of the challenges in managing a complex application infrastructure is ensuring that sensitive data remains secure. It is always important to store sensitive data, such as tokens, passwords, and keys, in a secure, encrypted form. In this lesson, we will talk about Kubernetes secrets, a way of securely storing data and providing it to containers. We will also walk through the process of creating a simple secret, and passing the sensitive data to a container as an environment variable.

- Relevant Documentation
https://kubernetes.io/docs/concepts/configuration/secret/

- Lesson Reference

- Create a secret using a yaml definition like this. It is a good idea to delete the yaml file containing the sensitive data after the secret object has been created in the cluster.

apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  myKey: myPassword

- Once a secret is created, pass the sensitive data to containers as an environment variable:

apiVersion: v1
kind: Pod
metadata:
  name: my-secret-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: myKey

- lesson-Secrets
-----------------

- kubernetes secrets: secrets are simply a way to store sensitive data in your kubernetes cluster. 

- So if you have a password or API token or certificate keys or things like that that are sensitive data that need to be stored in a secure, encrypted fashion, and you need to be able to pass those pieces of data to your applications that are running in your containers, then that's what Kubernetes secrets are for.
 
- Simply provides a way to securely store that information and then pass it to your containers at runtime so that they can access the sensitive information. 

- I'm gonna show you how to create a secret, and I'm also going to show you how to consume that secret inside a container through an environment variable. 
- So let's start by creating our secret, and I'm just going to create a yml definition of a secret.

$ vim my-secret.yaml
 
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  myKey: myPassword
 
* Note: 
  There are a lot of different ways to create kubernetes secrets, but we're just gonna do it through a yml definition file.   
 
- We need to specify API version. type of object is going to be secret, I want to give it a name. I'm going to use metadata name and I'll just call it my-secret. 

- In our case I'm going to just provide the raw data for my secret. So I'm going to set the stringData key That just allows me to provide raw unencoded data here and then I can enter basically as many key value pairs as I want. 

- You can actually store multiple values in a single secret object. We're just going to store one value, and I'm gonna make the key myKey. 
 
$ kubectl apply -f my-secret.yaml >> successfully created my secret . 

- Now, as you can see, this, yml file actually has the sensitive data right there in an unencrypted form. So just from best practices standpoint, it's a good idea for me to go ahead and delete that. So I'm just gonna delete my-secret.yml so that we no longer have the data stored in an insecure form on the server.
 
- Now I'm going to create my pod. And I'm just gonna call this one my-secret-pod. just a simple busybox pod called my-secret-pod. And what I'm gonna do is I'm going to provide this secret value as an environment variable to my container so that my container can use it. So if this was a password that my application was going to use to authenticate with some other service or something like that, it would be able to access that data using this environment variable. 

$ vim my-secret-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-secret-pod
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: myKey
          
          
- So I'm just going to specify the env key right here under the container spec to create a new environment variable, and I'll give it a name. And this is the actual environment variable name that will be visible to the container when it's running. So I'm just gonna call that MY_PASSWORD, and then I'll provide this valueFrom key. That's telling the cluster where it's going to get the value for this environment variable. And I will use secretKeyRef, which tells the cluster to look for this value in a secret and not only in a secret but in a specific key of that secret. 

- You may remember that our secret could have had multiple key value pairs if we wanted it to. So we're not only providing what secret this value is going to come from, but which key value pair within that secret. So I'm going to give it the name, and this is simply the name of the secret, which we just called my-secret. And then I need to provide the name of the key, which was just myKey. 

- So that's all I need to do in order to expose this environment variable containing the secret data to my container. 

$ kubectl apply -f my-secret-pod.yml >> pod was created successfully 
$ kubectl get pods >> pod is already up and running my secret pod right here. 

- So we were able to securely store our secret password and pass it into our container at runtime so that it could access that data as an environment variable.

