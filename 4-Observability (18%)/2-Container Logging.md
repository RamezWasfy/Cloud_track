# Container Logging

* When managing containers, obtaining container logs is sometimes necessary in order to gain insight into what is going on inside a container. Kubernetes offers an easy way to view and interact with container logs using the kubectl logs command. In this lesson, we discuss container logs and demonstrate how to access them using kubectl logs.

- Relevant Documentation
https://kubernetes.io/docs/concepts/cluster-administration/logging/

- Lesson Reference

- A sample pod that generates log output every second:

apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']

- Get the container's logs:

`kubectl logs counter`

- For a multi-container pod, specify which container to get logs for using the -c flag:

`kubectl logs <pod name> -c <container name>`

- Save container logs to a file:

`kubectl logs counter > counter.log`

- lesson- Container Logging
-----------------------------

- `Container logging:` now a containers normal consul output is going to go into something in kubernetes called the container log. And you can access the container logs using the kubectl logs command. 

- This is very useful. If you need to debug a container or if you just need to get a little more information about what's going on inside the container, you convey you that council output using the cubes, GTL logs command. So to show you how that works, 

- Now we create a pod called counter, essentially what this container is doing is it's running in a loop, and it's just counting and incrementing a number by one every second and outputting that number to the console. 

`vim counter-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']

- So if we run this container and look at the logs, we should see it is counting and incriminating by one every second. 

`kubectl apply -f counter-pod.yml` Pod is created. 

`kubectl logs counter` You can also specify the container with this -c flag and container_name. 

- But for this particular pod, it's not necessary to specify the container because this is a single container pod. There's only one container, so I don't actually need to spend by it. 

* Note: 
  if this was a multi container pod than when you're doing kubectl logs, you have to specify the container Otherwise, Kubernetes is not going to know which container you're trying to get logs for, and it's going to give an error message.

- As you can see, here is our list of log messages, and it has a timestamp for each one, and you can see it's incremental all the way from zero every second, and right now it looks like it's been running for about 137 seconds. That's just the log output that's coming from the console of my container, which I was able to access easily using the kubectl Command. 

- kubectl logs function is it prints the container logs as output. 

- If you want to save that output to a file or do something else with it, you're just going to use your normal linux command line bash tools in order to do that. 

`kubectl logs counter > counter.log`

- That's one of the great things about kubectl logs is it just outputs the contents of the container log and then you can manipulate that output or save that output to a file using the normal linux command line tools that you're used. 
