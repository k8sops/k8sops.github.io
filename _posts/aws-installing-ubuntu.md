---
layout: post
title:  "AWS Installing Ubuntu"
date:   2020-03-06 21:06:02 -0500
categories: kubernetes
---

### This is a virtualization-on-virtualization type of installation, and we can use these techniques to deploy to the major cloud providers.

#### Commands used in this lesson:

```
cat /etc/lsb-release

```

### Installing and testing Docker (remember to log out and back in for group changes to take effect):

```
sudo apt install -y docker.io
sudo usermod -aG docker cloud_user
docker run hello-world
Get and install Minikube:

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_1.4.0.deb
sudo dpkg -i minikube_1.4.0.deb
Configure and start Minikube:

sudo minikube config set vm-driver none
sudo minikube start
```

### Additional configurations and installation of kubectl:

```
sudo chown -R $USER $HOME/.kube $HOME/.minikube
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
sudo chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo kubectl create deployment --image nginx my-nginx
sudo kubectl expose deployment my-nginx --port=80 --type=NodePort
sudo minikube ip
sudo kubectl get svc
```