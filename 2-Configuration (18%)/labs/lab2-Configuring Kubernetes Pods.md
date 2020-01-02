# exercise - lab2 Configuring Kubernetes Pods

- Your company is nearing completion of their new mobile app, a candy-themed game. This application has some backend infrastructure the company plans to run using Kubernetes. They want to begin deploying one of their backend services to the cluster and have asked you to create a pod definition that meets the specifications required by the software. Create a pod definition in /home/cloud_user/candy-service-pod.yml, and then create a pod in the cluster using this definition to make sure it works.

- The specifications are as follows:

- The current image for the container is linuxacademycontent/candy-service:1. You do not need a custom command or args.
- There is some configuration data the container will need:
candy.peppermint.power=100000000
candy.nougat-armor.strength=10

- It will expect to find this data in a file at /etc/candy-service/candy.cfg. Store the configuration data in a ConfigMap called candy-service-config and provide it to the container as a mounted volume.

- The container will need to run with the file system group with the id 2000. You will need to set this using the securityContext.
- The container should expect to use 64MiB of memory and 250m CPU (use resource requests).
- The container should be limited to 128MiB of memory and 500m CPU (use resource limits).
- The container needs access to a database password in order to authenticate with a backend database server. The password is Kub3rn3t3sRul3s!. It should be stored in a secure fashion (as a Kubernetes secret called db-password) and passed to the container as an environment variable called DB_PASSWORD.

- The container will need to access the Kubernetes API using the ServiceAccount candy-svc. The service account already exists, so just configure the pod to use it.
If you get stuck, be sure to check out the task descriptions below or the solution video. Good luck!


# Solution

1- Create a ConfigMap called `candy-service-config` to store the container's configuration data.

$ vim candy-service-config.yml, containing the ConfigMap definition:

apiVersion: v1
kind: ConfigMap
metadata:
  name: candy-service-config
data:
  candy.cfg: |-
    candy.peppermint.power=100000000
    candy.nougat-armor.strength=10
    candy.lemon.acceptability=0

$ kubectl apply -f candy-service-config.yml

2- Create a Kubernetes secret called `db-password` to store the database password.

$ vim db-password-secret.yml to store the secret definition.

apiVersion: v1
kind: Secret
metadata:
  name: db-password
stringData:
  password: Kub3rn3t3sRul3s!

$ kubectl apply -f db-password-secret.yml

- Note that in a real-world scenario, you should delete this file after creating the object since it contains sensitive data!

3- Create the pod for the candy-service application according to the provided specification.

vim candy-service-pod.yml.

apiVersion: v1
kind: Pod
metadata:
  name: candy-service
spec:
  securityContext:
    fsGroup: 2000
  serviceAccountName: candy-svc
  containers:
  - name: candy-service
    image: linuxacademycontent/candy-service:1
    volumeMounts:
      - name: config-volume
        mountPath: /etc/candy-service
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-password
          key: password
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
  volumes:
  - name: config-volume
    configMap:
      name: candy-service-config


$ kubectl apply -f candy-service-pod.yml
$ Make sure your pod is up and running with kubectl get pod candy-service.