# ⚓ Kubernetes

Imagine you have a Docker container that is running an app that can be accessed externally. Suddenly, this application starts receiving a lot of traffic, so a new container with this application needs to be spun up so the traffic can be distributed between the two. This is where Kubernetes comes in as a container orchestration system. The term "orchestration" conjures up images of a literal orchestra. This metaphor does have some traction, with the containers being like instruments and Kubernetes being the conductor in control of the flow.&#x20;

<figure><img src="../../../../../.gitbook/assets/image (428).png" alt=""><figcaption></figcaption></figure>

* Kubernetes makes an application **highly available and scalable**. It does this by, for example, duplicating an application (and its db component) and load balancing external requests to this application to an available resource, therefore removing a single point of failure from the architecture and removing bottlenecks, which can slow down application response times. As mentioned earlier, scalability is a big concern for many companies; Kubernetes allows workloads to be scaled up or down to match demand.
* Kubernetes is also **highly portable** in that it can be run anywhere with nearly any type of infrastructure and can be used with a single or multi-cloud set, making it a very flexible tool.

## Cluster Architecture

### Kubernetes Pod

a pod as a group of one or more containers. These containers share storage and network resources. Because of this, containers on the same pod can communicate easily as if they were on the same machine whilst maintaining a degree of isolation.

<figure><img src="../../../../../.gitbook/assets/image (429).png" alt=""><figcaption></figcaption></figure>

### Kubernetes Nodes

Kubernetes workloads (applications) are run inside containers, which are placed in a pod. These pods run on nodes, there are two types:

The control plane (also known as "master node") and worker nodes. Both of these have their own architecture/components

(a cluster = a set of nodes)

## The Kubernetes Control Plane

The control plane manages the worker nodes and pods in the cluster.

#### Kube-apiserver

The API server is the front end of the control plane and is responsible for exposing the Kubernetes API.

#### Etcd

Etcd is a key/value store containing cluster data / the current state of the cluster.

#### Kube-scheduler

The kube-scheduler component actively monitors the cluster. Its job is to catch any newly created pods that have yet to be assigned to a node and make sure it gets assigned to one.

#### Kube-controller-manager

This component is responsible for running the controller processes.  One example of a controller process is the node controller process, which is responsible for noticing when nodes go down.

<figure><img src="../../../../../.gitbook/assets/image (430).png" alt=""><figcaption></figcaption></figure>

## Kubernetes Worker Node

Worker nodes are responsible for maintaining running pods.

#### Kubelet

Kubelet is an agent that runs on every node in the cluster and is responsible for ensuring containers are running in a pod.

#### Kube-proxy

Kube-proxy is responsible for network communication within the cluster. It makes networking rules so traffic can flow and be directed to a pod (from inside or outside the cluster). Traffic won't hit a pod directly but instead hit something called a Service (which would be associated with a group of pods), and then gets directed to one of the associated pods.

#### Container runtime

Pods have containers running inside of them. A container runtime must be installed on each node for this to happen.

<figure><img src="../../../../../.gitbook/assets/image (431).png" alt=""><figcaption></figcaption></figure>

## Communication Between Components

<figure><img src="../../../../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

## Daily interractions

Now we are going to see what we would be interacting with daily as a DevSecOps:

#### Namespaces

In Kubernetes, namespaces are used to isolate groups of resources in a single cluster.

#### ReplicaSet

As the name suggests, a ReplicaSet in Kubernetes maintains a set of replica pods and can guarantee the availability of x number of identical pods (identical pods are helpful when a workload needs to be distributed between multiple pods).

#### Deployments

Deployments in Kubernetes are used to define a desired state. Once this desired state is defined, the deployment controller (one of the controller processes) changes the actual state to the desired state. In the definition, you can note that you want this deployment to have a ReplicaSet comprising three nginx pods. Once this deployment is defined, the ReplicaSet will create the pods in the background.

#### StatefulSets

Stateful apps store and record user data, allowing them to return to a particular state. For example, suppose you have an open session using an email application and read 3 emails, but your session is interrupted. In that case, you can reload this application, and the state will have saved, ensuring these 3 emails are still read. Stateless applications, however, have no knowledge of any previous user interactions as it does not store user session data. For example, think of using a search engine to ask a question. If that session were to be interrupted, you would start the process again by searching the question, not relying on any previous session data.

Statefulsets enable stateful applications to run on Kubernetes, but unlike pods in a deployment, they cannot be created in any order and will have a unique ID (which is persistent, meaning if a pod fails, it will be brought back up and keep this ID) associated with each pod.![statefulset diagram](https://tryhackme-images.s3.amazonaws.com/user-uploads/6228f0d4ca8e57005149c3e3/room-content/3c0a1d5471d84bd4b86ae3e5a9fefa1f.svg)

<figure><img src="../../../../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

### Services

A service is placed in front of these pods and exposes them, acting as an access point. Having this single access point allows for requests to be load-balanced between the pod replicas. There are different types of services you can define: ClusterIP, LoadBalancer, NodePort and ExternalName.

<figure><img src="../../../../../.gitbook/assets/image (435).png" alt=""><figcaption></figcaption></figure>

#### Ingress

an application which can be made accessible by using a service to expose the pods running this application (let's call this service A). Imagine now that this web application has a new feature. This new feature requires its own application and so has its own set of pods, which are being exposed by a separate service (let's call this service B). Now, let's say a user requests to access this new feature of the web application; there will need to be some kind of traffic routing in place to ensure this request gets directed to service B. That is where ingress comes in. Ingress acts as a single access point to the cluster and means that all routing rules are in a single resource.

<figure><img src="../../../../../.gitbook/assets/image (436).png" alt=""><figcaption></figcaption></figure>
