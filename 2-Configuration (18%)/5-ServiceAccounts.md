# ServiceAccounts

* Kubernetes allows containers running within the cluster to interact with the Kubernetes API. This opens the door to some powerful forms of automation. But in order to ensure that this gets done securely, it is a good idea to use specialized ServiceAccounts with restricted permissions to allow containers to access the API. In this lesson, we will discuss ServiceAccounts as they pertain to pod configuration, and we will walk through the process of specifying which ServiceAccount a pod will use to connect to the Kubernetes API.


- Relevant Documentation
https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

- Lesson Reference

- Creating a ServiceAccount looks like this:

$ kubectl create serviceaccount my-serviceaccount

- Use the serviceAccountName attribute in the pod spec to specify which ServiceAccount the pod should use:

apiVersion: v1
kind: Pod
metadata:
  name: my-serviceaccount-pod
spec:
  serviceAccountName: my-serviceaccount
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]


- lesson-ServiceAccounts
-----------------

- Service accounts: They are something that allows your containers to actually interact with the kubernetes API itself, you may have some applications that actually need to be able to talk to the kubernetes cluster in order to do some automation or get information. 

- To do that securely, what you want to do is you wanna have service accounts that have limited permissions that allow those applications access only the portions of the API that they need to be able to access.
 
* Note for the exam: 
  For CKAD certificate, You don't really need to know too much about creating and configuring service accounts and roles and permissions themselves. But you do need to know how to instruct your pods to use a specific service account if they're going to interact with the kubernetes API So the way you can do that is simply by providing the name of that service account in the pod specifications. 
  
  
- In order for this to work, we need to actually have a really service account in our cluster that we can work with again. You don't need to focus too much on the process of creating the service account for the purposes of the exam. 

- kubectl create serviceaccount my-serviceaccount >> Service account has been created .

- Now i will use this service account in my pod which I'm going to call my-serviceaccount-pod. 

apiVersion: v1
kind: Pod
metadata:
  name: my-serviceaccount-pod
spec:
  serviceAccountName: my-serviceaccount
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    
    

-  I just need to add this attribute to the pod spec called serviceAccountName, and you'll remember our service account was just called my-serviceaccount. So that's all there is to it when you want to specify which service account to use within a pod. 

$ kubectl apply -f my-serviceaccount-pod.yml >> Pod is created
$ kubectl get pods >> Pod started up and everything is running

- We have successfully created a service account and configured our pod to utilize that service account. If it's going to need to interact with the kubernetes API