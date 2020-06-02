---
author: "Leoren Tanyag"
date: 2020-06-02
linktitle: Getting Started with Kubernetes
title: Getting Started with Kubernetes
categories: [  "Development", "howto"  ]
tags: ["k8s", "kubernetes", "ops"]
weight: 8
---


In this document, we will be learning about the basic components of kubernetes and how to create them.

### Prerequisites

##### We need a hypervisor - install VirtualBox
https://www.virtualbox.org/wiki/Downloads

*   This is something that we need inorder to run the nodes for our Kubernetes cluster, they can also be run on bare-metal, but since we will be running the nodes on our laptop, we will be running them on VMs.

##### Install kubectl
* k8s command line tool
* From: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos

```bash
brew install kubectl
```

##### Install Minikube
* Minikube is a Lightweight tool that allows you to run K8s locally. It uses a VM, and deploys a simple cluster containing 1 node.

```bash
brew install minikube
```
```bash
minikube start --vm-driver=virtualbox
```

*cleaning up:*
```bash
minikube stop
```

*delete local copy -*
```bash
minikube delete
```

## Cluster
Now, let us set up our Cluster.

*(A cluster is a set of node machines for run for running containerised application. If you are running Kubernetes = you are running a cluster)*

##### 1. If you haven't already, have VirtualBox running in the background.

*Note: what happens if its not running? - it picks a default hypervisor that it can find on your machine, which sometimes, you cannot manage or view, or it will error out. So it’s better to have it running and set “—vm-driver” to use this.*

##### 2. Start up minikube, specify virtualbox as your VM driver

```bash
minikube start --vm-driver=virtualbox
```

  * The above command
    * downloads Minikube ISO
    * sets up a local K8s environment via VirutalBox
 This creates a K8s cluster called “minikube”

Now have our cluster running.

![cluster](/img/getting-started-k8s/cluster.png)


## Namespaces
Namespaces are commonly used in large organisations, it provides a scope for Names (all objects in cluster must have Name and must be unique within the Namespace). It also provides a mechanism to attach authorisation and policy to that certain section of the cluster.

By default, k8s has the following namespaces:
* **default** - default k8s cluster initiated at cluster creation
* **kube-node-lease** - each node would have an associate Lease Object in this namespace.
    * lease object - a lightweight resource that improves the performance of node heartbeats when cluster scales
     * heartbeat - help determine the availability of a node
* **kube-public** - mostly reserved for cluster usage in case some resources should be visible publicly throughout the whole cluster
* **kube-system** - for objects created by k8s system


##### Creating a basic Namespace

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
     name: development
```

check:

```bash
kubectl get namespace
```

Now, we have a namespace called `deployment` in our cluster.

![namespace](/img/getting-started-k8s/namespace.png)

## Deployment
Here is where you can describe the desired state of your app's deployment. This gets read by the Deployment Controller which changes the actual state, to the desired state at a controlled rate.

##### Create a basic deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-all-deployment
  namespace: development #specify the namespace, otherwise it will be in default
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
        - name: helloworld
          image: karthequian/helloworld:latest
          ports:
            - containerPort: 80
```
In this yaml file, we are specifying that we want to run the `karthequian/helloworld:latest` image to `1 pod` to inside our `development` namespace. We are opening up `port 80` in our pod. The name of our deployment is `helloworld-all-deployment`.

*[`karthequian/helloworld:latest`](https://hub.docker.com/r/karthequian/helloworld/) is a simple nginx helloworld app for docker.*

##### Create deplyment

```bash
kubectl apply -f deployment.yaml
```
##### Check your pods
![deployment1](/img/getting-started-k8s/deployment1.png)

Here we can see that we have one pod running in that deployment.

Now we have a deployment running inside our `development` namespace.

![deployment2](/img/getting-started-k8s/deployment2.png)

## Service
Service is an abstract way to expose an application running on a set of Pods as a network service. You can access your deployment by exposing it as a service.

##### 1. Create your basic service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld-all-service
  namespace: development #specify the namespace, otherwise it will be in default
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  type: LoadBalancer
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: helloworld
```

##### 2. Create Service

```bash
kubectl apply -f service.yaml
```

##### 3. Check your service in the Cluster

![service](/img/getting-started-k8s/service.png)

##### 4. Get the URL (to check on the browser):

```bash
kubectl -n development get service
```

take note of the port IP under `PORT(S)`

```bash
minikube ip
```

![service2](/img/getting-started-k8s/service2.png)

this equals to something like:

```bash
http://<minikube IP>:<port>

#in our case its -

http://192.168.99.100:32320
```

paste on browser

![service3](/img/getting-started-k8s/service3.png)


#### Load balancing using Service

Now, if I increase the replicas to two, the deployment will spin up another pod. The service would then load balance the traffic between the two pods.

##### 1. Modify deplyment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-all-deployment
  namespace: development #specify the namespace, otherwise it will be in default
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
        - name: helloworld
          image: karthequian/helloworld:latest
          ports:
            - containerPort: 80
```

check:
![service4](/img/getting-started-k8s/service4.png)

Now we can see that we have two pods.

I opened up two browsers and hit the URL, and I can see that the traffic is being load balanced between the two pods:

![service5](/img/getting-started-k8s/service5.png)

Note: to get the above logs, I used a tool called `stern`. It lets you track pod logs.

You can download it by doing `brew install stern`, then run

```bash
stern -n development helloworld-all-deployment
```

Now we have a service that sits in front of our pods inside our `development` namespace.

![service6](/img/getting-started-k8s/service6.png)


## Horizontal Pod Autoscaler

HPA allows you to automatically scale the number of pods based on observed CPU utilisation.

In order for the hpa to work, we need to be able to see the usage metrics of our pods, we will need to enable **metrics-server**.

check your list of addons
```bash
 minikube addons list
```

enable metrics-server
```bash
 minikube addons enable metrics-server
```

For testing purposes, I’ve modified the deployment and scaled the number of pods (replica) I have to 1.
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-all-deployment
  namespace: development #specify the namespace, otherwise it will be in default
spec:
  selector:
    matchLabels:
      app: helloworld
  replicas: 1 # tells deployment to run 1 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
        - name: helloworld
          image: karthequian/helloworld:latest
          ports:
            - containerPort: 80
```

Now I have one pod.

![hpa](/img/getting-started-k8s/hpa.png)

Then I created my hpa.yaml
```yaml
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: helloworld-hpa
  namespace: development
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: helloworld-all-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 75

```
```bash
kubectl apply -f hpa.yaml
```

Check:

![hpa1](/img/getting-started-k8s/hpa1.png)

* With this HPA, I am tracking my pod's **CPU** usage and **Memory usage**
 * I allow up to 75% CPU utilization or 75% memory utilization before I scale up another pod
* I set up my minimum pod to two and max to 10. This will make the deployment scale up another pod even though my initial deployment only initially scaled one.

check:

![hpa2](/img/getting-started-k8s/hpa2.png)

Now, let's try putting some load on our server.

```bash
kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh

Hit enter for command prompt

while true; do wget -q -O- http://http://192.168.99.100:32320; done
```

If we want to speed it up, we might want to edit our hpa to only tolerate a lower level CPU load before it scales up. So I reduced it to 10% just for testing purposes.

![hpa3](/img/getting-started-k8s/hpa3.png)

Enventually, we can see that it scaled some new pods

![hpa4](/img/getting-started-k8s/hpa4.png)

Note that all newly created pods automatically go behind our **Service**, this gives us some kind of load balancing in place.

This is because when the HPA decided to scale, it changes the **replica** number on the actual **Deployment** configuration of the pod and the service knows that it load balances anything that is created with this deployment.

Now our architecture looks like:

![hpa5](/img/getting-started-k8s/hpa5.png)

## Ingress
Ingress manages external access to the service, typically via HTTP.
Allows you to expose HTTP and HTTPS routes from outside the cluster to **services** within the cluster. It adds an extra layer of routing and control.

##### Modify service.yaml
First we need to modify our service.yaml so that we don’t expose an external load balancer IP for it. I got rid of the “type: LoadBalancer”.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: helloworld-all-service
  namespace: development
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
      name: http
  selector:
    app: helloworld
```

before

![ing](/img/getting-started-k8s/ing.png)

after
![ing2](/img/getting-started-k8s/ing2.png)

##### Create ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: helloworld-ingress
  namespace: development
spec:
  rules:
    - host: mykube.com
      http:
        paths:
          - path: /
            backend:
              serviceName: helloworld-all-service
              servicePort: 80

```

I gave it a domain name `mykube.com` which points to my service `helloworld-all-service`.

Next, for testing purposes, because I haven’t actually bought the `mykube.com` domain, I’ve modified my `/etc/hosts` file to allow my laptop to recognise this domain, and have it point to my minikube cluster IP

```
192.168.99.100 mykube.com
```
*note: as mentioned above, you can get your IP using `minikube ip`*

Now, if I go on my browser, and type `mykube.com` I should be able to see my website.

![ing3](/img/getting-started-k8s/ing3.png)

Our architecture then looks like:

![ing4](/img/getting-started-k8s/ing4.png)
