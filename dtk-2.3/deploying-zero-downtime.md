---
layout: page
title: Deploying Zero Downtime
category: dtk-2.3
permalink: /dtk-2.3/discovering-services
chapter: 3
---

The desired state of our applications is changing all the time. The most common reasons for newstates are new releases. The process is relatively simple. We make a change and commit it to a coderepository. We build it, and we test it. Once we’re confident that it works as expected, we deployit to a cluster. It does not matter whether that deployment is to a development, test, staging, orproduction environment. We need to deploy a new release to a cluster, even when that is a single-node Kubernetes running on a laptop. No matter how many environments we have, the processshould always be the same or, at least, as similar as possible.

## Kubernetes Deployment
Just as we are not supposed to create Pods directly but using other controllers like ReplicaSet, weare not supposed to create ReplicaSets either. Kubernetes Deployments will create them for us.

```
cat deploy/go-demo-2-db.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
spec:
  selector:
    matchLabels:
      type: db
      service: go-demo-2
  template:
    metadata:
      labels:
        type: db
        service: go-demo-2
        vendor: MongoLabs
    spec:
      containers:
      - name: db
        image: mongo:3.3
        ports:
        - containerPort: 28017

```

* RS declaration earlier for DB is same as the above. The only differnce is the ***kind***
* We will regularly add --record to the kubectl create commands. This allows us to track each change to our resources such as a Deployments.

```
kubectl create -f deploy/go-demo-2-db.yml --record

kubectl get -f deploy/go-demo-2-db.yml

deployment.apps/go-demo-2-db created
```

* Describing the deployment

```
kubectl describe -f deploy/go-demo-2-db.yml

Name:                   go-demo-2-db
Namespace:              default
CreationTimestamp:      Fri, 31 Jan 2020 03:01:25 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
                        kubernetes.io/change-cause: kubectl create --filename=deploy/go-demo-2-db.yml --record=true
Selector:               service=go-demo-2,type=db
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  service=go-demo-2
           type=db
           vendor=MongoLabs
  Containers:
   db:
    Image:        mongo:3.3
    Port:         28017/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   go-demo-2-db-b449d94f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  109s  deployment-controller  Scaled up replica set go-demo-2-db-b449d94f to 1

```

* Upon observing the ***Events*** section we notice the deployment created a ReplicaSet.
* Listing all the objects:

```
kubectl get all

NAME                              READY   STATUS    RESTARTS   AGE
pod/go-demo-2-db-b449d94f-zc6kj   1/1     Running   0          5m4s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23d

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-db   1/1     1            1           5m4s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/go-demo-2-db-b449d94f   1         1         1       5m4s
```

* There is no difference between the use of Deployment vs ReplicaSet.

* Real advantage of Deployment is when we try to change of aspect of our deployment.

![alt text](images/deployment_stack.png "Deployment Stack")

