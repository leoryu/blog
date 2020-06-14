---
categories:
- Tech
date: "2019-06-23T20:26:32+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- kubernetes
- docker
title: Kubernetes in Docker
---

Installing Kubernetes locally is an annoying task. Without multiple bare machines, you need some extra storage and memory for VMs. Besides, you have to spend time on installation and dependency problems.If you want a Kubernetes cluster just for Dev/Test, it's a nightmare.

So here is a problem with Kubernetes, the Standard Kubernetes is great for production, but not suitable for building "temporarily". In order to find a solution, this paper will introduce a project, kind, which makes it possible to run Kubernetes cluster in Docker. 

<!--more--> 

## About kind

> kind is a tool for running local Kubernetes clusters using Docker container "nodes".

kind project pre-build a [kindest/node](https://hub.docker.com/r/kindest/node) Docker image and run it as Kubernetes nodes locally on a single machine. It helps people to build Kubernetes cluster easily without multiple bare machines or VMs.

If you want to build a Kubernetes cluster for Dev/Test, kind is the best choice at this time (2019).

## Installation and Basic Usage

Before enjoying kind, you need a Docker platform for running Kubernetes nodes and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) for accessing Kubernetes.

kind binaries are available in [releases page](https://github.com/kubernetes-sigs/kind/releases/). To install, download the binary for your platform and place it into your $PATH.

Using kind to create a Kubernetes cluster is very sample, all you have to do is running the following command:

```sh
kind create cluster
```

![kind create cluster demo](kind-demo.gif)

Then, according to the prompt, run:

```sh
export KUBECONFIG="$(kind get kubeconfig-path --name="kind")" && kubectl cluster-info
```

If you have seen your cluster info, congratulations, enjoy your Kubernetes travel.

## Multiple Nodes Cluster and Advanced Usage

If you run:

```sh
kubectl get nodes
```

You will find the cluster has just one node as control plane, because kind create 1-node cluster by default. If you want to build a multiple nodes cluster, you need to define a config file:

```yaml
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
nodes:
- role: control-plane
- role: worker
- role: worker
```

And run:

```sh
kind create cluster --config your/config/path --image kindest/node:v1.11.10 --name 3-nodes-cluster
```

Flag *--config* helps us to create a cluster with specific configuration, *--image* specify which image is used for Kubernetes node, and *--name* would give the cluster a friendly name.

Now run this command:

```sh
kind get clusters
```

You will get two clusters' name. Cluster kind is the default cluster, and Cluster 3-nodes-cluster is the cluster with three nodes which we just created.

Now run:

```sh
export KUBECONFIG="$(kind get kubeconfig-path --name="3-nodes-cluster")" && kubectl get nodes
```

You will see three nodes, one control plane and two workers.

To get more information about kind, you could check the [Official Documentation](https://kind.sigs.k8s.io/).