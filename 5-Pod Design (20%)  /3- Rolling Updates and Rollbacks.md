- One powerful feature of Kubernetes deployments is the ability to perform rolling updates and rollbacks. These allow you to push out new versions without incurring downtime, and they allow you to quickly return to a previous state in order to recover from problems that may arise when deploying changes. In this lesson, we will discuss rolling updates and rollback, and we will demonstrate the process of performing them on a deployment in the cluster.

- Relevant Documentation
https://v1-12.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment
https://v1-12.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment

- Lesson Reference

- Here is a sample deployment you can use to practice rolling updates and rollbacks.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-deployment
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
        image: nginx:1.7.1
        ports:
        - containerPort: 80
        
        
- Perform a rolling update.

`kubectl set image deployment/rolling-deployment nginx=nginx:1.7.9 --record`

- Explore the rollout history of the deployment.

`kubectl rollout history deployment/rolling-deployment`

`kubectl rollout history deployment/rolling-deployment --revision=2`

- You can roll back to the previous revision like so.

`kubectl rollout undo deployment/rolling-deployment`

- You can also roll back to a specific earlier revision by providing the revision number.

`kubectl rollout undo deployment/rolling-deployment --to-revision=1`

- You can also control how rolling updates are performed by setting maxSurge and maxUnavailable in the deployment spec:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-deployment
spec:
  strategy:
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
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
        image: nginx:1.7.1
        ports:
        - containerPort: 80
        
 
- lesson-Rolling Updates and Rollbacks
-----------------------------------------

- `Rolling updates and rollbacks` we've talked about deployments and how we can scale applications by changing the number of replicas that are specified in a deployment spec. But one of the other things we can do with deployments is rolling updates and rollbacks, and what they do is they give us some powerful tools for managing changes to our applications. 

- `Rolling updates` provide us a way to update a deployment to a new version of our container image by gradually updating replicas so that there's no down time. 

`Example:`

- Imagine that you have multiple replicas and we want update to a new version, normally we would just have to remove those replicas, change the version and then create new replicas. And there'd be some downtime while we're waiting for those new ones to spin up. 

- But what a rolling update does is it gradually rolls out the new version so it will spin up a few pods running the new version while leaving the old version continuing to run and only when the new version is ready, it will start removing those pods that are still running the old version of the container image. 

- we are gonna create a deployment here that we can use to test out our rolling updates. 

`vim rolling-deployment.yml`

apiVersion: apps/v1
kind: Deployment
metadata:
  name: rolling-deployment
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
        image: nginx:1.7.1
        ports:
        - containerPort: 80

1- Name of the deployment: rolling-deployment. 

2- The image we are using is `nginx:1.7.1` and we're going to do a rolling update to update that to a new version. 

3- `kubectl apply -f rolling-deployment.yml` >> deployment is now created and it looks like our 3 replicas are now up and running so let's go ahead and execute a rolling update. 

4- Use `kubectl set image` to change the image for our deployment and in order for the deployment to match that new desired state it is going to automatically execute a rolling update.
 
- So I'll just reference my deployment, which is deployment/rolling-deployment `kubectl set image deployment/rolling-deployment`

- Then I need to specify the container name, and the container that I wanna update was called nginx. And then I need to specify the image. Now, you remember we created the deployment with 1.7.1 So let's go ahead and just update to 1.7.9. 

`kubectl set image deployment/rolling-deployment nginx=nginx:1.7.9 --record`

- I am going to add this `--record` flag and what this does is it keeps track of the change that we're making with this commit It's gonna record information about this change that will allow us to do a rollback in the future if we need to.
 
`kubectl get pods -w` >> we still have our 3 running pods here, and there's changes occurring here. we have a new container creating that is a new pod that is running the new version of the code. And as you can see these new containers coming up. And as the new version is coming up and it's up and running, we start to see those old containers terminating and going away. 

- So it's gradually rolling out piece by piece the new code so that at the end we should have three completely new pods that are running this new image. 

`kubectl get pods` now we have three brand new pods. Now, if you look closely at the names for these pods, you'll notice those names are different than these three pods that we had here before. That's because these pods were terminated and replaced by these three new pods running the new version of the code.
 
- `Rollback` it essentially allows us to revert to a previous state. 

`Example` 

- let's say we did our rolling update and something went wrong. There's something wrong with our new code. Or maybe there was something wrong with our specifications. Maybe we specified the wrong version for a container image or something like that that causes our rolling update to not work and create some kind of problem and the easiest way to recover from that problem is to rollback, and that means to basically return to a previous state. 
 
`kubectl rollout history deployment/rolling-deployment` and what this does is it shows us a list of all the recorded changes to this particular deployment
 
- Revision 1: This is our starting state. So there's no change happened there.
 
- Revision 2: Under Change_CAUSE it gives us the command that we ran earlier as the cause for that change. So when we ran this command that created a new revision, which is revision 2 and that's the one that we're currently at right now. 

- So if we want to do a rollback, then we want to go back to revision one. 

- Another thing we can do is get more information about each revision. `kubectl rollout history deployment/rolling-deployment --revision=2` So I'm specifying a specific revision, and that's going to give me more information about that 2nd revision. 

- It shows me more information about the containers that were changed gives a little bit of information about the pod template. It's basically just a good way to look into each one of these revisions if you need a little more information about any of it.
 
- `kubectl rollout undo deployment/rolling-deployment` >> So now let's go ahead and execute a rollback. That's how we perform our rollback. So if I go ahead and execute this command as is, it's going to simply revert to the previous state. Revision 2 is the current state. 

- So if I just do kubectl rollout undo and reference the deployment, it's going to go back to the revision that's prior to the current one. So it's just going to go back to revision 1 so I can go ahead and execute that.

- Or i can also do this with `kubectl rollout undo deployment/rolling-deployment --to-revision=1` and that rolls back to a specific earlier revision. 

- So let's say that we created a problem and we don't want to go back to the previous revision. We want to actually go two or three revisions back. We can use that `--to-revision=` to do that. 


- `kubectl get pods` >> we can see that we actually have now 3 brand new pods. And what that means is we basically did the rolling update in reverse. The cluster created some new pods that are now running that nginx1.7.1 version again that we had to begin with and gradually using those to replace the ones from that revision 2 So we basically just did the same thing. 

- There is one other thing that I want to show you. I'm going to go ahead and edit our deployment that we created.
 
`kubectl edit deployment rolling-deployment` you can see the rolling update strategy. So here, under the spec, you'll see that there is a value called strategy and then rollingUpdate that's populated simply by the default values because we didn't give it a specific value when we created the deployment. 

- you'll notice we have these two values here called `maxSurge` and `maxUnavailable` 
 
- `maxSurge` this value determines the maximum number of extra replicas that can be created during a rolling deployment. We think about it. We've got three replicas. As soon as we start doing a rolling update, it's going to create some additional replicas running those new versions. So for a short period of time, there could potentially be more than three replicas. And the max surge simply puts a hard maximum on that, so you can use this, especially if you have a big deployment with a lot of different replicas. You can use that max surge value to control how fast or how gradually the deployment rolls out. You could set that very low, and it's going to just spend a few at a time and just gradually replace the replicas that Aaron your deployment. Or you could set that to a very high number, and it's going to do a huge chunk of them all at once, so it just gives you a little more control over how quickly those new instances are created. 

- `maxSurge` here is set to 25%. You can set it to a percentage. You can also set it to a specific number, so I could set it to 2 and say that the maximum is 2. Or I could just leave it as 25% which is going to be a percentage of that baseline number of replicas. 

- `maxUnavailable` the same way, it can be a number or percentage sign. What `maxUnavailable` does is it sets a maximum number on the number of replicas that could be considered unavailable during their rolling update. So if you're doing a rolling update or you're doing a rollback, you can use that value to ensure that at any given time, a large number or a small number of your active replicas are actually available. 

