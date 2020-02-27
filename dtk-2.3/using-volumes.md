---
layout: page
title: Using Volumes to Access Host's file system
category: dtk-2.3
permalink: /dtk-2.3/using-volumes
chapter: 8
---

* Most of the time, stateful applications store their state on disk. That leaves us with a problem. If acontainer crashes,kubelet will restart it. The problem is that it will create a new container based on the same image. All data accumulated inside a container that crashed will be lost.

* Kubernetes Volumes solve the need to preserve the state across container crashes. In essence,Volumes are references to files and directories made accessible to containers that form a Pod. The significant difference between different types of Kubernetes Volumes is in the way these files and directories are created.

* There are over twenty-five Volume types supported by Kubernetes. It would take us too much time to go through all of them. Besides, even if we’d like to do that, many Volume types are specific to a hosting vendor. For example,***awsElasticBlockStore*** works only with AWS, ***azureDisk*** and ***azureFile*** work only with Azure, and so on and so forth.

## Accessing Host’s Resources Through hostPath Volumes

### Use Case 1 : Docker client running in the Pod and Docker server to be used is on the Host (Kube Master)

```
kubectl run docker --image=docker:17.11 --restart=Never docker image ls
pod/docker created

kubectl get pods
NAME     READY   STATUS   RESTARTS   AGE
docker   0/1     Error    0          11s

kubectl logs docker
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

```

* The error above is due becoz of the missing Docker server in the Pod.
* By default, the client sends instructions to the server through the socket located in /var/run/docker.sock. We can accomplish our goal if we mount that file from the host into a container.
* Deleting the Pod for now and proceeding to fix:

```
kubectl delete pod docker
pod "docker" deleted
```