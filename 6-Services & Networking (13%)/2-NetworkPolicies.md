# NetworkPolicies

- From a security perspective, it is often a good idea to place network-level restrictions on any communication between different parts of your infrastructure. NetworkPolicies allow you to restrict and control the network traffic going to and from your pods. In this lesson, we will discuss NetworkPolicies and demonstrate how to create a simple policy to restrict access to a pod.

- Relevant Documentation
https://kubernetes.io/docs/concepts/services-networking/network-policies/

- Lesson Reference

- In order to use NetworkPolicies in the cluster, we need to have a network plugin that supports them. We can accomplish this alongside an existing flannel setup using canal:

`wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml`

`kubectl apply -f canal.yaml`

- Create a sample nginx pod:

apiVersion: v1
kind: Pod
metadata:
  name: network-policy-secure-pod
  labels:
    app: secure-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    

- Create a client pod which can be used to test network access to the Nginx pod:

apiVersion: v1
kind: Pod
metadata:
  name: network-policy-client-pod
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    
    
- Use this command to get the cluster IP address of the Nginx pod:

`kubectl get pod network-policy-secure-pod -o wide`

- Use the secure pod's IP address to test network access from the client pod to the secure Nginx pod:

`kubectl exec network-policy-client-pod -- curl <secure pod cluster ip address>`

- Create a network policy that restricts all access to the secure pod, except to and from pods which bear the allow-access: "true" label:

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          allow-access: "true"
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          allow-access: "true"
    ports:
    - protocol: TCP
      port: 80
      

- Get information about NetworkPolicies in the cluster:

`kubectl get networkpolicies`
`kubectl describe networkpolicy my-network-policy`

lesson - NetworkPolicies
--------------------------

`Network policies`  allows you to control what traffic is allowed to come in and out of your pods. You can imagine this is very useful from a security stand point because it allows you to restrict what's allowed to talk to your pods and what your pods are allowed to talk to.

- Please note that by default, all pods in the cluster can communicate with any other pod, and reach out to any available IP
 
- In order to test out network policy is we're gonna need to do a couple of things to set up our cluster. So far, we've been using just the flannel networking plug in and out of the box. Flannel does not support network policies by itself, so we're gonna have to install an additional plugin in order to get that to work. 

- For CKAD exam, everything you need is already gonna be installed for you. You're just gonna have to work with the network policies in the cluster. 

- So this is really just if you want to follow along using a playground server. 

`wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml` >> this is going to install something called Canal, which is going to give us that network policy support. 

`kubectl apply -f canal.yaml` >> this install canal, and I should have support for network policies in my cluster. 

- Create a sample nginx pod: `vi network-policy-secure-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: network-policy-secure-pod
  labels:
    app: secure-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

- I'm gonna create a couple sample pods. We're gonna use these pods to test our network policy. And the 1st one is going to be called network-policy-secure-pod.
  
- This is a simple pod that's listening on Port 80. And it has this label called app equals secure-app. So take note of that label because we're going to be using that in a selector in just a moment. 

`kubectl apply -f network-policy-secure-pod.yml` >> this will create the nginx pod.

- Then I have a second pod called network-policy-client-pod. 

`vi network-policy-client-pod.yml`

apiVersion: v1
kind: Pod
metadata:
  name: network-policy-client-pod
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]


- In order to test our network policy, what we're gonna do is we're gonna use this pod as a client, and we will attempt to access that network-policy-secure-pod nginx from inside of this pod, so we're gonna use this pod to access the other pod, and that's how we'll test our network interaction. 

`kubectl apply -f network-policy-client-pod.yml` >> create the second pod
`kubectl get pods` >> looks like both of my pods are up and running

- I'm going to get a little more information about my secure pod. 

`kubectl get pod network-policy-secure-pod -o wide` >> Now i am looking for the cluster IP and i am going to use that IP to access the pod.  `returns 10.244.1.2`
 
- So we're gonna do `kubectl exec network-policy-client-pod -- curl 10.244.1.2` and this command simply allows us to run a command inside of the client pod and we're just gonna do a curl on the cluster IP of our secure pods that were accessing one pod from inside the other pod. 

- I got a response and It's just the nginx welcome page, So I was successfully able to contact the other pod from inside my client pod. And that's because we don't have our network policy set up yet. 

- By default, without any network policies, your pods are going to be completely open. Anything can talk to them and they can talk to anything else. So, as you can imagine, that's not great for security in a lot of scenarios. 


- Now let's go ahead and build a network policy that will allow us to restrict that access. 

`vim my-network-policy.yml`

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          allow-access: "true"
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          allow-access: "true"
    ports:
    - protocol: TCP
      port: 80

- So I need to specify an `apiVersion`. Now, network policies are part of the networking API. So it's `networking.k8s.io/v1` 

- For the `kind`. I'm just going to specify `NetworkPolicy` and let's give it a `name` and We'll just call it my `my-network-policy`. 

- Now I'm ready to provide the `spec`. So the first thing I want to do is I want to go ahead and add `podSelector`. And what this does is it lets me provide a selector that is going to determine which pods this policy applies to. 

- By default, pods are completely open and they remain completely open until a network policy selects them. So as soon as a pod becomes selected by a network policy, it's access is going to be controlled by that network policy or potentially multiple network policies. If there are more than one that select the same pod.
 
- You may remember our secure pod had this label called `app equals secure-app`. So we're gonna go ahead and provide this label here, and that means that this policy is now going to select that nginx pod that we created earlier. And once we create this policy in the cluster. All access is no longer going to be open to that pod It's going to be restricted by this network policy. 

- Network policies work based on a whitelist model. That means as soon as a network policy applies to a pod In other words, it selects a pod using the pod selector. That pod is completely locked down, nothing can talk to it. And it cannot talk to anything until we provide some rules that will whitelist specific traffic in and out of the pod. 

- So I'm going to provide a list of `policyTypes`, and this is gonna tell the cluster which type of traffic this policy applies to. So we have `Ingress` and `Egress` as the two possible values here.
 
- Ingress simply means traffic coming into the pod.
- Egress means traffic leaving or going out of the pod. 

- So if I were to take away the egress and just have ingress, that means that this policy would only apply to incoming traffic so incoming traffic would be locked down. But traffic leaving the pod would be completely open because neither this policy nor any other network policies in the cluster currently apply to outgoing traffic on the pod.
 
- Now we're ready to go ahead and actually provide our rules. This is where we whitelist certain types of traffic coming in and out of the pods. 

- I'm gonna provide `ingress`. And here's where we provide our ingress rules, which again control incoming traffic. And I put in a ray here and then I can specify `from` so from means we're referring to the sources of the traffic. So we're talking about traffic coming into the pod from these places and we're gonna whitelist certain sources there are out now here. 

- I'm going to provide a `podSelector`. There's different types of selectors that we can provide here. Right now, we're just gonna focus on podSelector which selects a group of pods, and we'll use a `matchLabels` selector and let's just make it `allow-access` equals `true`. So what this means is this selector applies to pods that have this label called allow access, and the value is true and essentially what this rule does is any pods that do not have this label in this value will not be allowed because they won't meet this whitelist condition.
 
- But if the pod that is trying to contact our secure pod does have this label in that sore spot and it's set to true, then that traffic is gonna be whitelisted and the rule will allow the traffic to come through. Now, four are from section. You can see this is level with our rule that specified by from We also need to specify a list of ports.
 
- And I'm going to provide the `protocol`, which is `TCP`, and the `port` which will simply be port `80`. So all we're doing is we're allowing traffic to come in from pods that have this label and that traffic also has to be specifically on port 80. All other traffic will be blocked by this rule. 

- Now I want to show you how to do outgoing rules They're almost exactly the same as incoming rules. 

- So I just put them here under this `egress` key and instead of listing from i will list `to`. So this is specifying rules for traffic that is coming out of our pod and is going to a certain destination. And in order for that traffic to be open, it has to match one of the whitelist statements that is provided here underneath, too. And in fact, underneath the to pretty much everything in terms of the syntax here is exactly the same as it was for our ingress rule. 

- And what we're doing is we're basically opening up traffic, both coming in from pods that bear this label and traffic that is going out to pods that bear that same label. 

`kubectl apply -f my-network-policy.yml` >> our network policy is now in the cluster

`kubectl get networkpolicies` >> you can even see the pod selector here. So it's very easy to see just at a glance which pods these network policies apply to. 

`kubectl describe networkpolicy my-network-policy` >>  You can see not only the pod selector, but you can also see the rules here, and that gives you some insight into what each of these network policies is actually doing. 

- So now that we've created our network policy, I'm just going to re run this command that we ran earlier. This is just that command that's going to try to access our secure pod from inside the client pod.

`kubectl exec network-policy-client-pod -- curl 10.244.1.2` >> I'm running that command and it does not seem to be completing. In fact, it seems like it's probably gonna time out. 

- The reason why it was timing out is simply because our network policy is now blocking access. Only Pods that have this allow access equals true label are going to be able to communicate with our secure pod.
 
- Now lets try to allow our client pod to access this pod by going to the pod and provide that `allow-access: "true"` label. So I'm just going to edit our client pod 

`kubectl edit network-policy-client-pod` >> Looks like there's not a labels field yet, so I'll come here under metadata and specify labels. And then I will add this label, allow-access and in quotes i will set that to True. 

- So now I have updated my client pod to add that label and let's see what happens if we rerun this curl statement. 

`kubectl exec network-policy-client-pod -- curl 10.244.1.2` >> we're successfully able to access the other pod because now this client pod meets the whitelist condition that is set by the network policy and therefore it is allowed access. 

- Now, before we end the lesson. There is one other thing that I want to talk about and that is the `selector types for ingress and egress rules`.
 
* `podSelector`: This allows for specifying a pod selector. Traffic from/to pods matching the selector will match the rule.

* `namespaceSelector`: This is similar a podSelector, but allows you to select one or more namespaces instead. Traffic from any pods in the matched namespaces will match the rule. However, if you use podSelector and namespaceSelector together, the matched pods must also reside in one of the matched namespaces.

* `ipBlock`: This allows you to specify a range of IP addresses, using CIDR notation. Sources/destinations that fall within the IP range will match the rule. You can also list IPs to exclude from the range using except.

There's actually three different types of selectors.  We used a pod selector, which allows us to control access according to a set of pods that are defined by a selector such as matching labels. We can also use a `namespaceSelector`. 

- We can have multiple namespaces. You can actually put labels on namespaces and then select them with the namespace selector. So this can be useful if you have different teams using different namespaces and you want to grant access across certain namespaces but not others You can use the namespace selector to do that 

- then you have the `ipBlock` selector, which lets you specify a range of IPs instead of a label or a pod, and that one is especially useful if you're trying to control access to and from external resources outside the cluster because those resources don't have pods. You're not gonna be able to use a pod selector to refer to them, but you can limit it using an `ipBlock`

