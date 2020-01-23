---
layout: page
title: chap 1 - Scaling with Replicasets
category: dtk-2.3
permalink: /dtk-2.3/chap1
---

# Scaling Pods with ReplicaSets

* Its primary, and pretty much only function, is to ensure that a specified number of replicas of a Pod matches the actual state (almost) all the time.
*  ReplicaSets make Pods scalable
* We can think of ReplicaSets as a self-healing mechanism. As long as elementary conditions are met (e.g., enough memory and CPU), Pods associated with a ReplicaSet are guaranteed to run. They provide fault-tolerance and high availability.
* ReplicaSet is the next-generation ReplicationController. ReplicaSet has support for selectors and ReplicationControllers didn't.

## Creating a ReplicaSet
```
apiVersion:  apps/v1
kind: ReplicaSet
metadata:
  name: go-demo-2
spec:
  replicas: 2
  selector:
    matchLabels:
      type: backend
      service: go-demo-2
  template:
    metadata:
      labels:
        type: backend
        service: go-demo-2
        db: mongo
        language: go
    spec:
      containers:
      - name: db
        image: mongo:3.3
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: localhost
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
```
