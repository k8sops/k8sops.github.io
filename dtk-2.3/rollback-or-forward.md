---
layout: page
title: Rollingback or Forward
category: dtk-2.3
permalink: /dtk-2.3/rollback-or-forward
chapter: 4
---

# Rollingback or Forward

## Undo Rollouts
* There are scenarios where rolling forward is not an option. In those cases we can rollback using the ***rollout undo*** command.

```
kubectl rollout undo -f deploy/go-demo-2-api.yml

kubectl describe deploy/go-demo-2-api.yml
```
* Looking at the events we see that the process started increasing the replicas of the older ReplicaSet and decreasing those from the latest ones.


***Past rollouts:***
```
kubectl rollout history -f deploy/go-demo-2-api.yml 

deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
2         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
```

***Going back to a certain revision:***
If we were do rollout out few more versions of the code, there can be a use case to rollback to a certain revision in the rollout history.
```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:3.0 --record

kubectl rollout status -f deploy/go-demo-2-api.yml 
deployment "go-demo-2-api" successfully rolled out

kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:4.0 --record
deployment.apps/go-demo-2-api image updated

kubectl rollout history -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
2         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
4         kubectl set image api=vfarcic/go-demo-2:3.0 --filename=deploy/go-demo-2-api.yml --record=true
5         kubectl set image api=vfarcic/go-demo-2:4.0 --filename=deploy/go-demo-2-api.yml --record=true
```

* Now say we would like go back to revision #2, which is the image tag version 2.0

```
kubectl rollout undo -f deploy/go-demo-2-api.yml --to-revision 2
deployment.apps/go-demo-2-api rolled back

kubectl rollout history -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api 
REVISION  CHANGE-CAUSE
3         kubectl create --filename=deploy/go-demo-2-api.yml --record=true
4         kubectl set image api=vfarcic/go-demo-2:3.0 --filename=deploy/go-demo-2-api.yml --record=true
5         kubectl set image api=vfarcic/go-demo-2:4.0 --filename=deploy/go-demo-2-api.yml --record=true
6         kubectl set image api=vfarcic/go-demo-2:2.0 --filename=deploy/go-demo-2-api.yml --record=true
```

* revision #6 is now tag version 2.0 and, ***if this was the “real” application running in a production cluster, our users wouldcontinue interacting with the version of our software that actually works.***

## Rolling back failed deployments
Other than a bad deployment there are other cases that needs a rollback. Cases like deployment failures, where say Pods didn't get created.

```
kubectl set image -f deploy/go-demo-2-api.yml api=vfarcic/go-demo-2:does-not-exists --record
deployment.apps/go-demo-2-api image updated


cloud_user@anirban1c:~/k8s-specs$ kubectl get pods -l type=api
NAME                             READY   STATUS         RESTARTS   AGE
go-demo-2-api-586448d5dd-gxksl   0/1     ErrImagePull   0          58s
go-demo-2-api-586448d5dd-rft67   0/1     ErrImagePull   0          58s
go-demo-2-api-6c8867677b-5zj5n   1/1     Running        1          4h27m
go-demo-2-api-6c8867677b-t5dnb   1/1     Running        1          4h27m

kubectl rollout status -f deploy/go-demo-2-api.yml 
error: deployment "go-demo-2-api" exceeded its progress deadline
```

But in real pipeline we would like to automate the process of detecting a failed deployment other than waiting for time out and checking ***status***.

Fortunately the ***status*** command return a non-zero return.
```
echo $?
1

kubectl rollout undo -f deploy/go-demo-2-api.yml 
deployment.apps/go-demo-2-api rolled back

kubectl get pods -l type=apiNAME                             READY   STATUS    RESTARTS   AGE
go-demo-2-api-6c8867677b-5zj5n   1/1     Running   1          4h35m
go-demo-2-api-6c8867677b-q7jb5   1/1     Running   0          9s
go-demo-2-api-6c8867677b-t5dnb   1/1     Running   1          4h35m
```

The ***undo*** corrected the situation now.

## Merging definitions in a Single file


```
cat deploy/go-demo-2.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-db
  labels:
    type: db
    service: go-demo-2
    vendor: MongoLabs
spec:
  selector:
    matchLabels:
      type: db
      service: go-demo-2
  strategy:
    type: Recreate
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

---

apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-db
spec:
  ports:
  - port: 27017
  selector:
    type: db
    service: go-demo-2

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-2-api
  labels:
    type: api
    service: go-demo-2
    language: go
spec:
  replicas: 3
  selector:
    matchLabels:
      type: api
      service: go-demo-2
  template:
    metadata:
      labels:
        type: api
        service: go-demo-2
        language: go
    spec:
      containers:
      - name: api
        image: vfarcic/go-demo-2
        env:
        - name: DB
          value: go-demo-2-db
        readinessProbe:
          httpGet:
            path: /demo/hello
            port: 8080
          periodSeconds: 1
        livenessProbe:
          httpGet:
            path: /demo/hello
            port: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: go-demo-2-api
spec:
  type: NodePort
  ports:
  - port: 8080
  selector:
    type: api
    service: go-demo-2
```

```
kubectl create -f deploy/go-demo-2.yml --record --save-config
deployment.apps/go-demo-2-db created
service/go-demo-2-db created
deployment.apps/go-demo-2-api created
service/go-demo-2-api created
```

All four objects(two Deployments and two Services) were created, and we can move on and explore ways to update multiple objects with a single command.


## Updating Multiple Objects
