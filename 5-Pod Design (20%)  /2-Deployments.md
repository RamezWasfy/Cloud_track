# Deployments

- Deployments provide a variety of features to help you automatically manage groups of replica pods. In this lesson, we will discuss what deployments are. We will also create a simple deployment and go through the process of scaling the deployment up and down by changing the number of desired replicas.

- Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

- Lesson Reference

- Here is a simple deployment for three replica pods running nginx.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
        
        
- You can explore and manage deployments using the same kubectl commands you would use for other object types.

`kubectl get deployments`

`kubectl get deployment <deployment name>`

`kubectl describe deployment <deployment name>`

`kubectl edit deployment <deployment name>`

`kubectl delete deployment <deployment name>`
 
- lesson-Deployments
------------------------

- Deployments are where some of the powerful automation features of kubernetes really start to shine in terms of managing our pods. 

- Deployments provide a way to declaratively manage a dynamic set of replica pods. So we essentially specify a desired state for a set of pods. And the cluster is going to constantly work to maintain that desired state by automatically creating, removing and modifying the replica pods accordingly. They provide powerful functionality such as scaling and rolling updates.

- Here we have a deployment that has a pod template and a selector. The deployment automatically manages a set of pods that match the pod selector and uses the deployment template to determine the desired state of those pods. So let's just jump right in and create a deployment. 

- So I'm going to create a deployment called nginx-deployment. So I'll just make an object descriptor here, and we'll create this deployment just like we create any other kubernetes object. 

`vim nginx-deployment.yml`

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

- The `API version`. Now, deployments are part of the apps API. So we need to specify that we're gonna put apps/v1 .
- For the `kind` we will put deployment we need to give it a name and we'll call it nginx-deployment.
- The next thing we need to do is create a specifications for the deployment. `spec` 

1- `replicas` replicas tells the deployment how many pods we want to be part of this deployment. So I'm going to go ahead and put 3. That means that we're gonna have three replica pods. That's gonna be the desired state. So as soon as we create our deployment, it's going to look and see that we have 0 pods and it will immediately create three pods in order to ensure that the cluster matches that desired state. once we've created the deployment, we can come back and change the number of replicas and the deployment will automatically create or remove pods to match that new replica number. 

2- `selector` selectors provide us a way to select a list of pods based upon their labels. So this selector is going to determine which pods are considered to be part of this deployment. 

- The deployment will automatically manage any pods that match the selector that we provide here. So I'm going to provide a `matchLabels` selector, which means we're looking for pods that match a particular label. And we'll just make that label called `app equals nginx`. So we're gonna have an nginx app on this deployment is going to match any pods that have that app label with a value of in nginx.
 
- `template` This is a pod template that will be used to create and manage the pods that are part of this deployment. So this is a desired state for our pods. And when new pods are created, they're going to be created according to this template. Now, what goes under this template is simply a pod descriptor.
 
- Here i need to specify `metadata` for our pod, and I need to specify `labels` here. And these are the labels that are going to be part of the pods that are created by the deployment. remember, the deployment manages the pods that match this selector. So it's very important to make sure that the labels that we add to our pod template here match the selector. Otherwise, that deployment is going to create pods, and then it won't be able to manage them because they won't actually match the matchLabels under the selector. 

- So we're going to specify the labels to `app: nginx`. So any pods that are created by this deployment are gonna have that label. Therefore, they're going to match the selector, and therefore the deployment will be able to manage them. 

- `pod spec` we need our list of containers. I'm just gonna create one here with the name of nginx and for the image I will use `nginx:1.7.9`  And, of course, nginx listens on Port 80. So assuming that we want to actually use this as some kind of Web server will need to expose that port using our `containerPort` of 80. 

`kubectl get pods` >> You can see that we don't have any pods right now, at least in the default name space.
 
`kubectl apply -f nginx-deployment.yml` >> the deployment is created in. 

`kubectl get pods` >> we can see that we have 3 pods up and running. So we requested three replicas and now we have our 3 pods. 
 
`kubectl get deployments` >> give us information about deployments we have and I can see my nginx deployment and this desired count up to date and available represents the replicas. So we asked for three replica. So that's the desired number through your current. That means they match the current specifications and three are available. That means they're up and running and ready to service requests. 

- So now I want to show you what happens if we go ahead and edit our deployment.
 
`kubectl edit deployment nginx-deployment` and I can come down here to the specifications, and that i can actually change the number of replicas. So let's change that to four and see what happens. 

`kubectl get pods` >> We already have 1/4 pod. wait some time till they become 4/4

