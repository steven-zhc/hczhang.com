---
title: Oracle Linux Docker User's Guide
date: 2017-01-15 20:03:24
tags: 
  - docker
  - oracle linux
---

# Oracle Linux Docker User's Guide

[Offical Reference Document](https://docs.oracle.com/cd/E52668_01/E75728/html/section_kfy_f2z_fp2.html)

# Update system kernel

The following commonds will run as root
> sudo su -

Add proxy
```bash
cat > /etc/profile.d/proxy.sh <<- EOF
export http_proxy=PROXY_HOST
export https_proxy=PROXY_HOST
export no_proxy=localhost,127.0.0.1
EOF
```

Get the public-yum.ol7.repo
```bash
cd /etc/yum.repos.d
curl -o public-yum-ol7.repo  http://public-yum.oracle.com/public-yum-ol7.repo 
```

Disable other repository source file
```bash
mv EPEL-ol7.repo EPEL-ol7.repo.bak
mv ULN-Base-ol7.repo ULN-Base-ol7.repo.bak
```

Edit the public-yum-ol7.repo file and change (enabled=) for each block {0 to disable 1 to enable}
> vi public-yum-ol7.repo

-disable everything
-enable ol7_UEKR4

Upgrade system kernel to **Unbreakable Enterprise Kernel Release 4 (UEK R4)**
> yum upgrade kernel*


Restart system using new kernel
> init 6

Check System Kernel
> uname -a


# Install docker engine

> vi /etc/yum.repos.d/public-yum-ol7.repo

- -enable ol7-latest
- -enable ol7-addons
- -disable everything else

Install Docker
> yum install docker-engin


# Docker Configuration

To config web proxy
> [Service]
> Environment="HTTP_PROXY=proxy_URL:port"
> Environment="HTTPS_PROXY=proxy_URL:port"

Flush changes
> systemctl daemon-reload


# Docker Service

Start Docker daemon
> systemctl start|restart docker


Start Docker at boot
> systemctl enable docker


# Non-root users to run docker cmds

Create the docker group
> groupadd docker

Restart docker service
> service docker restart


Add the users
> usermod -a -G docker user1

