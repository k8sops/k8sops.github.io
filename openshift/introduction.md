---
layout: page
title: Introduction
category: openshift
permalink: /openshift/intro
chapter: 1
---

![](images/flavors.png)

![](images/stack.png)

![](images/toolchain.png)

![](images/openshift_components.png)

![](images/setup_options.png)


## Projects

* Uses Kubernetes namespaces underneath for isolation between projects.

![](images/intro/projects.png)

* User types:
![](images/intro/user_types.png)

Take notice of the prefix for service & system users.

* OAuth settings:
![](images/intro/oauth.png)

```
oc get projects
oc get users
oc adm policy add-cluster-role-to-user cluster-admin administrator
```

## Builds and deployments

![](images/intro/deployment.png)

#### Build Strategies:
1. Dockerfile

2. S2I

3. 
