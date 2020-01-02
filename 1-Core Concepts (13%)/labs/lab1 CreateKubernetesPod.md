Your company is getting ready to launch a new website, and they need you to set up an nginx web server in their Kubernetes cluster. The nginx server will need to be accessible via network in the future, so you will need to expose port 80 as a containerPort for the nginx container. Your team has also asked you to ensure that nginx runs in quiet mode for the time being to cut down on unnecessary log output. You can do this by setting the command to nginx and passing the following arg to the container: -g daemon off; -q. As this nginx server belongs to the Web team, you will need to create it in the team's web namespace.

To summarize:

* Use the nginx container image.
* The container needs a containerPort of 80.
* Set the command to nginx
* Pass in the -g daemon off; -q args to run nginx in quiet mode.
* Create the pod in the web namespace.

- Once the pod is created, you should be able to find it with kubectl get pods -n web. Once the pod is created, you can get more information about its current status with kubectl describe pod nginx -n web.



Lab Solution : 

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: web
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["nginx"]
    args: ["-g", "daemon off;", "-q"]
    ports:
    - containerPort: 80
    
    