# Boilerplate for running node.js services in a scalable way using Kubernetes

Not only is this a boilerplate, but it's also a commit for commit instruction on how everything is built from the ground up.
The gateway to entry is to spend an hour of your time to go through the sylabus, at the end you will be a well-educated slinger of containers.

## 0. Kubernetes

Kubernetes is an intelligent container runtime. It takes care of ensuring that runtime execution policy is enforced for workloads on the compute cluster.
It also provides a number of incredibly useful ancillary discovery and management features.

There are 3 main components in any kubernetes architecture.

### The Cluster

This is the meatgrinder that takes the bits of data submitted by customers and turns it into the beautiful sausage on offer.

These are the bare-metal or virtual servers that run the kubernetes `kublet` process.
This process manages the cluster and reacts to control instructions by monitoring the API server.

Google uses [CoreOS](https://coreos.com/) for the VMs that are your cluster members.
CoreOS is an easily updateable stripped-down derivative of Gentoo Linux that is intended explicitly for this purpose.

The cluster also provides an overlay network for inter-pod communication and a DNS system for workload discovery.

On Google Container Engine (GKE) the overlay network allocates a unique IP-per-pod, a configuration that I highly recommend even if you don't run on GKE.
This allows taking advantage of the DNS and network protocols to handle discovery which is very useful as you can eschew the need to manage workload port allocations.

### The Container Runtime

Kubernetes uses the container runtime to effect workload isolation and quota control.
Containers are used by developers to increase productivity, mitigate dependency hell, and ease deployment.

Kubernetes supports running workloads on either the docker or rkt container engines. In our system we'll be using docker.

Besides the runtime itself, the container registry is also a very important component.
This is where workload images are stored and where kubernetes retreives them for execution.

### Workloads

#### Directory Structure

Keeping an organized directory structure for managing the scripts that define your cluster, workloads, and policies will pay many efficiency dividends.

__services/__: In this directory are all of the files for defining each service in our infrastructure.
This is a good place to store things like Dockerfiles, configuration, and secrets; actual apptlication source and build files are best left in a discrete repository.


## 1. The Compute

We're going to start off with a more immediate compute resource, the computer you're on now using a tool called `minikube`. 

> Don't let the title of the article fool you, kubernetes can be run many places.
> On bare-metal, on AWS, on Azure, on DigitalOcean, on Linode, and on-laptop.
>
> The reason for the article title is because I believe Google has simply pulled off the best Cloud 3.0 offering.
> They have integrated kubernetes cluster components as native management entities and provide in-depth support for pods running on their infrastructure.

### Minikube

[Minikube](https://github.com/kubernetes/minikube/releases) is a simple stand-alone binary, on Linux this is the installation process:

```console
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.18.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

Once installed, a simple incantation `minikube start` rewards with the following:

```console
$ minikube start
Starting local Kubernetes cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.
```

### Kubectl

The final line there hints at the only other tool that you'll need to interact with your kubernetes cluster, `kubectl`.
[Kubectl](https://kubernetes.io/docs/tasks/kubectl/install/) is a simple to use command line client to the kubernetes cluster API.

Everything in kubernetes revolves around the API, itself having some unique features.
It uses good development practices in exposing a consistent interface of entities, properties, and actions.
But one of the more unusual features is a robust subscription system.

The components of the kubernetes cluster subscribe to entities and entity collections on the API.
Add/change/remove events from these subscriptions are used as policy by the k8s services.
All execution and configuration are driven by managing entities via the k8s API.

I'm a proponent of local control of authorization tokens and then projecting authority across infrastructure when executing a command.
Dormant sessions and insecure authentication tokens can cause many security headaches.
For that reason, I tend to use `kubectl` from my workstation as a primary cluster management tool outside of continuous delivery pipelines.

It's helpful to keep the entity approach in mind when working with `kubectl`. For example, to fetch a list of all pods running on the cluster use:

```console
$ kubectl get pods
No resources found.
```

We have no pods! That may not be a big deal to you if you have no idea what a pod is. A pod is an isolated execution environment.
We'll look at some of the more advanced use-cases of pods in the future, but for now we can think of them as being synonymous with a workload.
