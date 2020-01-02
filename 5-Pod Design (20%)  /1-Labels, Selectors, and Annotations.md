# Labels, Selectors, and Annotations

* Kubernetes labels provide a way to attach custom, identifying information to your objects. Selectors can then be used to filter objects using label data as criteria. Annotations, on the other hand, offer a more free form way to attach useful but non-identifying metadata. In this lesson, we will discuss labels, selectors, and annotations. 

- Relevant Documentation
https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/

- Lesson Reference

- Here is a pod with some labels.

apiVersion: v1
kind: Pod
metadata:
  name: my-production-label-pod
  labels:
    app: my-app
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx
    
- You can view existing labels with kubectl describe.

`kubectl describe pod my-production-label-pod`

- Here is another pod with different labels.

apiVersion: v1
kind: Pod
metadata:
  name: my-development-label-pod
  labels:
    app: my-app
    environment: development
spec:
  containers:
  - name: nginx
    image: nginx
    
- You can use various selectors to select different subsets of objects.

`kubectl get pods -l app=my-app`

`kubectl get pods -l environment=production`

`kubectl get pods -l environment=development`

`kubectl get pods -l environment!=production`

`kubectl get pods -l 'environment in (development,production)'`

`kubectl get pods -l app=my-app,environment=production`

- Here is a simple pod with some annotations.

apiVersion: v1
kind: Pod
metadata:
  name: my-annotation-pod
  annotations:
    owner: terry@linuxacademy.com
    git-commit: bdab0c6
spec:
  containers:
  - name: nginx
    image: nginx

- Like labels, existing annotations can also be viewed using kubectl describe.

`kubectl describe pod my-annotation-pod`

- lesson- Labels, Selectors, and Annotations
---------------------------------------------

* labels, selectors and annotations 

- `Labels:` are key value pairs that we can attach to kubernetes objects, and they're used to identify those objects in some way. There are multiple different features of kubernetes that use labels to select a group of objects that they need to interact with. And we can attach labels to objects using the metadata.labels section of the object descriptor. 

`Example:`

`vim my-production-label-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: my-production-label-pod
  labels:
    app: my-app
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx

- In this simple pod description, I can add labels under metadata just with this labels key and then I can add whatever labels I want here so I'll just add one called `app equals my-app` that will identify what application this pod is part of. And I'll add another one called `environment equals production`.
 
- Now we could make another pod that has `environment equals development` and use that label to kind of differentiate between those two pods, so we could have one that's meant to run in our production environment. Another one's meant to represent our development environment on our label allows us to differentiate between those two.

`vim my-development-label-pod.yml`
 
apiVersion: v1
kind: Pod
metadata:
  name: my-development-label-pod
  labels:
    app: my-app
    environment: development
spec:
  containers:
  - name: nginx
    image: nginx
 

`kubectl apply -f my-production-label-pod.yml` >> my-production-label-pod has been created.

`kubectl apply -f my-development-label-pod.yml` >> my-development-label-pod has been created.
 
`kubectl get pods --show-labels` >>  we can actually see the labels that are associated with this object. 

`kubectl describe pod my-production-label-pod` this will also allow me to see those existing labels  If I look at the output from describe, there's the label section, The existing label calls for an object. 
 
# How labels can be used
 
- We can use labels as part of selectors. 

`Selectors:` they identify and select a specific group of objects using the object labels. 
 
- When you do `kubectl get`, you can actually provide a selector, and the command will return the list of pods that matches that selector. And these selectors are the same selectors that are used for those other features of kubernetes like services, deployments and network policies. 

`Example`

`kubectl get pods -l` and now I'm gonna provide a selector first, type of selector is an `equality based selector` you may remember that the two pods that we created earlier had the label `app equals my-app`. 

- So if I run this selector, what this is going to do is it's going to select pods that have the app label where the value of that label equals my-app. So this should select our two pods that we created earlier because they both have that label. 

`kubectl get pods -l app=my-app` >> These returns the 2 pods that we have just created with app=my-app label


- Now change the label, select `environment=production`, you may remember that one of our pods had `environment=production` The other one was `environment=development`. So they both had the environment label. They have different values for that label. 

`kubectl get pods -l environment=production` >> I only get the production pod

`kubectl get pods -l environment=development` >> I only get the development pod

- I can also use `inequality based selector` >> All I have to do is just put an exclamation mark ! before the equal sign. So I'm saying environment is not production and what this is going to do is it should select that development pod because it's the one where the value does not equal production.

`kubectl get pods -l environment!=production` >> I only get the development pod

- I can also use `set based selectors` >>  i am going to put single quotes here because set based selectors have spaces. So I need to put that in quotes and to do a set based selector, I do the label name, which is environment and then in then I'm gonna put parentheses and I can provide a list of possible values. 

`kubectl get pods -l 'environment in (development,production)'` >> this will select all pods were environment equals any of the values between () , this returns our 2 pods So I'm basically selecting any pod where this label is any of these values. 

- We can also provide `multiple selectors` 

`Example`

`kubectl get pods -l app=my-app,environment=production` >> this actually return our production pod.


`annotations:`

- Annotations are very similar to labels. They look very similar. They're basically just key value pairs, just like labels are and they're used to store custom metadata about objects.
 
- However, annotations are a little different from labels because they are not intended to be identifying, and annotations cannot be used as part of a selector. So they're basically just a way to attach your own custom metadata to objects. But you cannot use them to select those objects.
 
- Annotations are very useful if you're working with external tools or automation, because if you need to attach some kind of custom data field to any of your kubernetes objects, annotations allow you to do that. 

`vim my-annotation-pod.yml` >> 

apiVersion: v1
kind: Pod
metadata:
  name: my-annotation-pod
  annotations:
    owner: terry@linuxacademy.com
    git-commit: bdab0c6
spec:
  containers:
  - name: nginx
    image: nginx


- So I'm just gonna create a pod called my-annotation-pod. just like we would do with labels we can put labels under metadata. We can also so put our annotations under metadata. So I'm going to add an annotation called owner. And let's say that this annotation is basically the contact point for this object. So the person who I need to call if I have a question about this pod and I'll just put that terry@linuxacademy.com 

- This is just an example of one way you could use annotations. You could use it to specify the owner of an object I'm gonna do also a git commit, and this is the kind of thing you would do if you were doing some kind of automation. If somehow this pod was linked to a git repositories somewhere, and you had some automation that was creating or updating the pod. You could actually provide the get commit that's associated with that action. So just kind of an example of some of the types of metadata that you could attach using annotations. 

`kubectl apply -f my-annotation-pod.yml` >> pod is now created

`kubectl describe pod my-annotation-pod` >> you can see the annotations here