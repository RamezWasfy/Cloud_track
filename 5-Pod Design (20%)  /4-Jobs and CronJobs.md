# Jobs and CronJobs

- Kubernetes provides the ability to easily run container workloads in a distributed cluster, but not all workloads need to run constantly. With jobs, we can run container workloads until they complete, then shut down the container. CronJobs allow us to do the same, but re-run the workload regularly according to a schedule. In this lesson, we will discuss Jobs and CronJobs and explore how to create and manage them.

- Relevant Documentation
https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

- Lesson Reference

- This Job calculates the first 2000 digits of pi.

apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
  
  
You can use kubectl get to list and check the status of Jobs.

`kubectl get jobs`

- Here is a CronJob that prints some text to the console every minute.

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
          
          
- You can use kubectl get to list and check the status of CronJobs.

`kubectl get cronjobs`

# Lesson - Jobs and CronJobs
-------------------------------

- Jobs and CronJob: Jobs are essentially a way to reliably execute a workload until it completes. 

- Jobs are similar to pods, because both jobs and pods are designed to run a container. But most of the time pods are used to run containers that are intended to run constantly. So something like a Web server or some type of service, they're designed to be constantly running however jobs are a little different. A job runs a container that does some work, and when that work is finished, the container shuts down and it doesn't continue to run. If we want to run a particular workload and we want to automatically restart if it fails and just make sure that it runs once and succeeds, then a kubernetes job is a great way to accomplish that. 

- Now let us create a job in the cluster, we call the job pi . This job is going to simply calculate a certain number of the digits of pi,

`vim pi.yml`

apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4

- Just like the other types of kubernetes objects, and I'm going to specify my API version. Now jobs are part of the batch API. So I'm going to list `API version` `batch/v1` and for the `kind` I'll just put `Job`. 

- We can give it a name with metadata name and I'll just call it pi. Now we can go ahead and provide our job specifications. And for jobs, we create a pod template very similar to what we do with deployments. This is basically a pod specifications that is part of the job specifications.
 
- So our pod specifications needs a pod spec and just like we would have in any other pod specifications, we need our list of containers, so I'll just create `one container with name of pi`
- For the `image`, I'll just do `perl`. And that's just a simple image that's able to run perl. And we will execute a command that's going to calculate the digits of pi using perl. 

- So I'll do perl as my first part of my command, this is just a simple perl statement that's going to calculate. We'll say the 1st 2000 digits of pi. So now I've got my command and I'm going to set a `restartPolicy`, and I'm going to set this to `never` because this is a job we wanted to execute once and then stop so we don't need it to restart. 

- Now i am going to set a `backoffLimit` to 4. And what the backoffLimit does is it determines how many times this job will attempt to run again if it fails, and basically, by setting it to 4, it means that if it fails four times in a row, it's not going to continue to try to run. And that just prevents us from continuing to try to run something that's broken over and over and over when it's not going to succeed. 

 `kubectl apply -f pi.yml` >> This will create our job in the cluster.
 `kubectl get jobs` >> We can see the status 0/1 completions looks like it's still running.
  
- This is going to take a while to calculate those digits of pi. it looks like it's done Now.
 
`kubectl get pods` >> you can actually see that there is a pod associated with this job. 

- So it is very important to know that what the job really does, is it creates a pod behind the scenes. But instead of a normal pod that's constantly running, it's a pod that runs and when it's finished, it goes into the completed status, which means the container is no longer running, so the pod still exists. It's still an object in Kubernetes, but the container is actually completed. 

- So I've successfully created a job. Now if I wanted to get the output of my calculation of pi >> `kubectl logs POD_NAME` and there's the output from my job. 



`CronJobs`: CronJobs are very similar to regular jobs, but essentially what they do is they allow you to execute jobs on a schedule. 

- If you're familiar with CronJobs in linux, it's basically the same thing. A CronJob in linux allows you to specify what is called a cron expression that determines when and how often the job will be executed. And whenever that time comes around, it's going to go ahead and run whatever is part of that cron expression.
 
- In kubernetes, a CronJobs is the same thing. We provide a cron expression that specifies that schedule, and on that schedule it's going to execute that job. So a job allows us to run a workload once until it's completed. However CronJobs allows us to run a job workload according to a schedule. 

- So let's go ahead and create a CronJob. I'm just gonna call it Hello. 

`vi hello.yml`

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

- I'll create my specifications, as always with the API version. Now, just like jobs. CronJobs are part of the batch API, but they're not in version 1, so we actually have to specify batch/v1beta1.
 
- In order to get our CronJob to work next, I'm going to specify the `kind` which is just `CronJob` and our `metadata`, where I'll give it the `name` of `hello` and then we have a CronJobs specifications. 

- Now the first thing here is the `schedule` attribute, which is where we provide our actual cron expression that's going to determine when and how often our CronJob is going to execute . If you're not all that familiar with cron expressions, that's fine. Just be aware that basically what they do is they allow you to define a schedule that's going to determine when something is going to execute. 

- So I'm gonna go ahead and enter a cron expression here, and what this expression does is it's going to tell this CronJob to run every minute. So once a minute this CronJobs is going to execute. Now I can provide a jobTemplate. And as you can imagine, the jobTemplate is where we basically provide a job spec. So that job spec that we created earlier in this lesson we're now going to put one of those right here under the job template. 

- So we need a spec, and now we have another template, and this is the pod template that is inside of the job template. So I got a lot of nested templates and specs here. But what's going on is we have our cron job spec. Then we have a template for our job and a specifications for the job. Then we have a template for the pod and a specifications for the pod. 

- we need our containers. And I'll just make one `container` called `Hello` and we'll just do the `busybox` image and I'll just provide a list of arguments. We will call a shell Command with /bin/sh. And then what will do is we will print out the date and just echo `Hello from the Kubernetes cluster`.
 
- So what this cron job is going to do is every minute it's going to print a time stamp and then this statement Hello from the Kubernetes cluster. Now I'm going to go ahead and create a `restartPolicy`, and this is part of the pod specifications. So in terms of indentation, it's level with my containers, and I'll set this one too, `OnFailure`, which just means it will be automatically restarted if it fails. But if it completes then it won't be restarted, and I'll save that. 

`kubectl apply -f hello.yml` >> create the job. Then I could just do a cube. 

`kubectl get cronjob` and we can see the status our cronjob. Last schedule is the last time it ran, which means currently has not actually run yet. So if I run it again, we can see that it's currently active. That means that right now it is currently running. 

`kubectl get pods` >> We should be able to see the pod, and it looks like it's already completed. 

`kubectl get cronjob` >> we can see it is not active. And it last started 32 seconds ago. So it looks like our cron job has run once and completed and it ran successfully in one minute. It should run again. But if we want to get the output of it, we can do it the same way we did for our pi job with `kubectl logs POD_NAME` we can see are time stamped and hello from the kubernetes cluster. 

- So in this lesson, we've talked about how to create jobs and KRON jobs. Jobs and KRON jobs are powerful tools if you have workloads that you want to run in the cluster either once or on a schedule.
