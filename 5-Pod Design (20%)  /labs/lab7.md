# Configuring CronJobs in Kubernetes

Description

- Pods are not the only way to run workloads in Kubernetes. You can also use jobs to execute something once, or you can use cron jobs to execute workloads on a schedule. This lab provides an opportunity to learn about cron jobs by implementing a simple scheduled job in a working Kubernetes cluster.

# Additional Information and Resources

- Your company has a simple data cleanup process that is run periodically for maintenance purposes. They would like to stop doing this manually in order to save time, so you have been asked to implement a cron job in the Kubernetes cluster to run this process. Create a cron job called cleanup-cronjob using the linuxacademycontent/data-cleanup:1 image. Have the job run every minute with the following cron expression: */1 * * * *


# Solution

- Create the cron job in the cluster

- Create a descriptor for the cron job with `vi ~/cleanup-cronjob.yml`

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cleanup-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: data-cleanup
            image: linuxacademycontent/data-cleanup:1
          restartPolicy: OnFailure
          
- Create the cron job in the cluster.

`kubectl apply -f ~/cleanup-cronjob.yml`

- Allow the cron job to run successfully

- Give the cron job a minute or so to run once, and then check the status.

`kubectl get cronjob cleanup-cronjob`

- You should see a timestamp under LAST-SCHEDULE, indicating the job was executed.
  
