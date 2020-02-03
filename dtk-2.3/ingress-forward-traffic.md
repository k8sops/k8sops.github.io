---
layout: page
title: Ingress Forward Traffic
category: dtk-2.3
permalink: /dtk-2.3/ingress-forward-tarffic
chapter: 6
---

Ingress objects manage external access to the applications running within the Kubernetes clusters. We would need features like ***forwarding rules*** on paths and domains and ***SSL termination***

A tradiotinal setup would probably have a external load balancer or a proxy. ***Ingress*** provides an API to do all these things and other features that would be required for  a dynamic cluster.

## Exposing defincies with access using Kubernetes Services