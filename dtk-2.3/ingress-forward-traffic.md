---
layout: page
title: Ingress Forward Traffic
category: dtk-2.3
permalink: /dtk-2.3/ingress-forward-tarffic
chapter: 6
---

Ingress objects manage external access to the applications running within the Kubernetes clusters. We would need features like ***forwarding rules*** on paths and domains and ***SSL termination***

A tradiotinal setup would probably have a external load balancer or a proxy. ***Ingress*** provides an API to do all these things and other features that would be required for  a dynamic cluster.

## Exposing defeciences with access using Kubernetes Services

* With just services we need to know the port where the app is listening. For end user that wouldn't be a good user experience.

```
curl localhost:31961/demo/hello
hello, world!
```

* Creating the setup first:

```
kubectl create -f ingress/go-demo-2-deploy.yml 

deployment.apps/go-demo-2-db created
service/go-demo-2-db created
deployment.apps/go-demo-2-api created
service/go-demo-2-api created

kubectl get -f ingress/go-demo-2-deploy.yml

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-db   1/1     1            1           41s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
service/go-demo-2-db   ClusterIP   10.98.219.81   <none>        27017/TCP   41s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/go-demo-2-api   3/3     3            3           41s

NAME                    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/go-demo-2-api   NodePort   10.107.125.129   <none>        8080:31444/TCP   40s

kubectl get pods

NAME                             READY   STATUS    RESTARTS   AGE
go-demo-2-api-86469df75d-jqt7c   1/1     Running   0          2m13s
go-demo-2-api-86469df75d-ll9sg   1/1     Running   0          2m12s
go-demo-2-api-86469df75d-ql8gz   1/1     Running   0          2m12s
go-demo-2-db-5bfdb95844-zdlsj    1/1     Running   0          2m13s

```

* Now to access the services this our option:

```
PORT=$(kubectl get svc go-demo-2-api -o jsonpath="{.spec.ports[0].nodePort}")

echo $PORT
31444

curl -i "http://localhost:$PORT/demo/hello"
HTTP/1.1 200 OK
Date: Fri, 07 Feb 2020 15:37:46 GMT
Content-Length: 14
Content-Type: text/plain; charset=utf-8

hello, world!
```

* This approach might not look to be a bad option with One App. To increase the issue we will deploy one more app:

```
kubectl create -f ingress/devops-toolkit-dep.yml --record --save-config

deployment.apps/devops-toolkit created
service/devops-toolkit created

kubectl get -f ingress/devops-toolkit-dep.yml

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/devops-toolkit   3/3     3            3           83s

NAME                     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/devops-toolkit   NodePort   10.104.41.21   <none>        80:30714/TCP   83s

PORT=$(kubectl get svc devops-toolkit -o jsonpath="{.spec.ports[0].nodePort}")

echo $PORT
30714

curl -i "http://localhost:$PORT"

```

* So we got a new PORT for the new App.

![alt text](images/accessing-apps-thru-services.png "Accessing Apps thru Services")

A user sends a request to one of the nodes of the cluster. That request is received by a Service and load balanced to one of the associated Pods. It’s a bit more complicated than that, with iptables, kube DNS, kube proxy, and a few other things involved in the process.

## What is the ideal setup?

* There should a single route without port. Something like this:

```
for the hello app ->
curl "http://$IP/demo/hello"

for the devopstookit api ->

curl -H "Host: devopstoolkitseries.com" http://localhost
curl: (7) Failed to connect to localhost port 80: Connection refused

```
* Still doesn't work. So Services can give what we need.
* Last,but not least, we should be able to make some, if not all, applications(partly) secure by enabling HTTPS access. That means that we should have a place to store our SSL certificates. We could put them inside our  applications, but that would only increase the operational complexity. Instead, weshould aim towards SSL offloading somewhere between clients and the applications, and it should come as no surprise that Kubernetes has a solution for all these.

## Enabling Ingress Controllers

* We need a mechanism that will accept requests on pre-defined ports (e.g.,80 and 443) and forward them to Kubernetes Services. It should be able to distinguish requests based on paths and domains as well as to be able to perform SSL offloading.
* Out of the Box Kubernetes does not have a solution for Ingress Controllers. The Controllers that come in built in the kube-controller-manager library doesn't have support for Ingress.
* Instead of the Controller, kube-controller-manager offers a ***Ingress Resource*** that other third-party solutions can can utilize to provide requests forwarding and SSL features. In other words, Kubernetes only provides an API, and we need to set up a Controller that will use it.