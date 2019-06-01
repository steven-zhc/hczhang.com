---
title: kubernetes intro day 1
date: 2019-05-13 16:24:28
tags: [k8s, devops, docker]
---

# Initializing the Master

Calico networking plugin

```bash
kubeadm init --pod-network-cidr 192.168.0.0/16 \
--service-cidr 10.96.0.0/16 \
--apiserver-advertise-address $(ifconfig ens160 | grep 'inet addr'| cut -d':' -f2 | awk '{print $1} ')
```

# Calico Project

```bash
kubectl apply -f http://docs.projectcalico.org/v2.3/getting- started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

# Tips

```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
```