# Monitoring Applications

- Relevant Documentation
https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

- Lesson Reference

- Here are some sample pods that can be used to test kubectl top. They are designed to use approximately 300m and 100m CPU, respectively.

apiVersion: v1
kind: Pod
metadata:
  name: resource-consumer-big
spec:
  containers:
  - name: resource-consumer
    image: gcr.io/kubernetes-e2e-test-images/resource-consumer:1.4
    resources:
      requests:
        cpu: 500m
        memory: 128Mi
  - name: busybox-sidecar
    image: radial/busyboxplus:curl
    command: [/bin/sh, -c, 'until curl localhost:8080/ConsumeCPU -d "millicores=300&durationSec=3600"; do sleep 5; done && sleep 3700']
    
    
apiVersion: v1
kind: Pod
metadata:
  name: resource-consumer-small
spec:
  containers:
  - name: resource-consumer
    image: gcr.io/kubernetes-e2e-test-images/resource-consumer:1.4
    resources:
      requests:
        cpu: 500m
        memory: 128Mi
  - name: busybox-sidecar
    image: radial/busyboxplus:curl
    command: [/bin/sh, -c, 'until curl localhost:8080/ConsumeCPU -d "millicores=100&durationSec=3600"; do sleep 5; done && sleep 3700']
    
    
- Here are the commands used in the lesson to view resource usage data in the cluster:

`kubectl top pods`

`kubectl top pod resource-consumer-big`

`kubectl top pods -n kube-system`

`kubectl top nodes`

- lesson- Monitoring Applications
----------------------------------

- Monitoring applications: we will focus on checking the resource usage of our pods using the kubectl top command. 

- Now i will create 2 pods.
 
 1- resource-consumer-big. This is just a pod that's designed to consume a certain amount of CPU. Specifically, this pod is designed to consume 300 milli cores or 300m CPU. 

`vim resource-consumer-big.yml`
  
apiVersion: v1
kind: Pod
metadata:
  name: resource-consumer-big
spec:
  containers:
  - name: resource-consumer
    image: gcr.io/kubernetes-e2e-test-images/resource-consumer:1.4
    resources:
      requests:
        cpu: 500m
        memory: 128Mi
  - name: busybox-sidecar
    image: radial/busyboxplus:curl
    command: [/bin/sh, -c, 'until curl localhost:8080/ConsumeCPU -d "millicores=300&durationSec=3600"; do sleep 5; done && sleep 3700']
    
      
`kubectl apply -f resource-consumer-big.yml`>> pod is now created. 

2- resource-consumer-small. this pod is just using fewer resources instead of using 300 CPU, this one's going to use about 100. 

apiVersion: v1
kind: Pod
metadata:
  name: resource-consumer-small
spec:
  containers:
  - name: resource-consumer
    image: gcr.io/kubernetes-e2e-test-images/resource-consumer:1.4
    resources:
      requests:
        cpu: 500m
        memory: 128Mi
  - name: busybox-sidecar
    image: radial/busyboxplus:curl
    command: [/bin/sh, -c, 'until curl localhost:8080/ConsumeCPU -d "millicores=100&durationSec=3600"; do sleep 5; done && sleep 3700']

`kubectl apply -f resource-consumer-small.yml`>> pod is now created. 

`kubectl get pods` 2 pods are up and running.
 
`kubectl top` >> to check the resource usage of these pods. 
 
- You also have to specify the resource type because you have two options, that are pods and nodes.
  
`kubectl get pods` Here are my two pods that are in the default namespace. 

- Just like every other kubectl command. We're always working with the namespace. And if we don't specify the namespace, it's going to do default namespace. So we're looking in the default namespace where our two pods are and as you can see, my big consumer is using about 300, the small consumer is using about 99 cores.
 
- I can also see the memory usage. They're using about 15 megabytes each. I can also check the usage of a specific pod. So it is kubectl top pods POD_NAME. 

`kubectl top pods resource-consumer-small` 

- You can also specify pods in another namespaces. 

`kubectl top pods -n kube-system` >> this is all of our system pods with all of the resource usage information for each one of them.

`kubectl top nodes` this is to get information about the usage on each one of my nodes. 

- This gives the current usage and percentage. So that means that this node is currently using about 6% of its overall available CPU and 29% of its overall available memory. 