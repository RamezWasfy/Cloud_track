# Installing Metrics Server

* In order to practice using some of the monitoring features in Kubernetes, you will need a running metrics server in your cluster. It is not necessary to know how to install the metrics server in order to pass the exam, but you will need to install it in your practice cluster if you are following along. In this lesson, we will go over to quickly and easily install the metrics server in your practice cluster.

- Clone the metrics server repo and install the server using kubectl apply:

`cd ~/`

`git clone https://github.com/linuxacademy/metrics-server`

`kubectl apply -f ~/metrics-server/deploy/1.8+/`

- Once you have installed the metrics server, you can use this command to verify that it is responsive:

`kubectl get --raw /apis/metrics.k8s.io/`

- lesson- Installing Metrics Server
------------------------------------

- For CKAD exam. You don't need to know how to install the metric server. 

- However, in order to follow along in the next lessons, you're going to need to have a metric server running in your cluster.
 
- Metric server: It provides an API that gives you access to additional data about your pods and notes some of the more useful data surrounds resource usage such as CPU and memory usage within different parts of your cluster. 

- So metric server is very powerful tool if you want to get access to information about your resource usage or build automation around that data.
 
`Metric server installation:`

`cd ~/` >> go to home dir

`git clone https://github.com/linuxacademy/metrics-server` >> clone the metric server Repo

`kubectl apply -f ~/metrics-server/deploy/1.8+/` >> I will go into the deploy directory and then 1.8 plus because we are using a version of Kubernetes that's New Earth in 1.8 I'm just gonna run that kubectl apply. And that's going to create a variety of different objects in my cluster. 

`To test metric server functionality` 

`kubectl get --raw /apis/metrics.k8s.io/` >> make sure that the metric server is up and running and that the metrics API is responsive.

- So an easy way to do that is to use kubectl get --raw >> And what that means is I'm going to access one of the APIs directly and I'm just doing this to make sure that the metrics API is working and the raw API that I'm gonna access is just /apis/metrics.k8s.io and I ran that command. 
 
- Voila I got a response >> this  indicates that my metrics API is up and running and it's responsive. Now take note that once the metric server is up and running, it can take a few minutes before you start getting actual metrics data about some of your objects. 

- The metric server collects that data periodically. So if you're playing around in exploring the metrics API don't be surprised if you don't have any data yet for a pod that you just created. 

- If that happens, just wait a couple minutes and you'll start getting data for that pod. 