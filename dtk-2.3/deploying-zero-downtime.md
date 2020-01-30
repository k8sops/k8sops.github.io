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
