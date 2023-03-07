---
title: Production Reference Architecture for the Rancher Stack
author: Zack Brady - Field Engineer, Andy Clemenko - Field Engineer
contact: zack.brady@rancherfederal.com, andy.clemenko@rancherfederal.com
---

# Production Reference Architecture for the Rancher Stack

![rgs-banner](/images/rgs-banner-rounded.png)

## Table of Contents

* [About Us](#about-us)
* [Introduction](#introduction)
* [Rancher Stack](#Rancher-Stack)
* [Rancher Deployment Strategy](#Rancher-Deployment-Strategy)
* [Final Thoughts](#final-thoughts)

## About Us

A little bit Zack Brady, his history, and what he's done in the industry.

* DOD/IC Contractor
* U.S. Military Veteran
* Open-Source Contributor
* Built and Exited a Digital Firm
* Volunteer Firefighter/EMT

A little bit Andy Clemenko, his history, and what he's done in the industry.

* DOD/IC Contractor
* Docker Federal (PS, SE, TAM)
* RedHat/Stackrox Federal (SE)
* Rancher Advocate (6+ Years)
* Volunteer Firefighter/EMT/Pump Operator

## Introduction

It's almost impossible to configure and implement a solution to fit every customer, every environment, and every situation. Hopefully this reference architecture will be able to provide you a great starting point to apply to your customer, your environment, and your situation. This guide is intended to highlight an idea deployment of Rancher and a few downstream clusters.

By the end of this guide we will have a very good sense of what a production deployment of Rancher looks like. Use this guide to build your clusters or compare to what has already been built. This guide is solely looking at the operating system up. Meaning we do include tips for virtualized environments like affinity rules.

Let's start by looking at the Rancher stack.

## Rancher Stack

There are other great products in the catalog, but we are going to focus on the "trinity" for this guide. The Rancher basic stack looks like the following.

* Operating System of Choice - We prefer RPM based due to Selinux support.
* [RKE2](https://www.rancher.com/products/rke) - Rancher's Kubernetes Distribution
* [Longhorn](https://www.rancher.com/products/longhorn) - Rancher's Persistent Storage
* [Rancher](https://www.rancher.com/products/rancher) - Multi-Cluster Kubernetes Manager

Let's look at the Rancher Deployment Strategy before we talk specifics.


## Rancher Deployment Strategy

![spoke](/images/topo.jpg)

The good news is that there is quite a bit of [documentation](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/rancher-deployment-strategy) on the two Rancher Deployment Strategies. We are going to focus on the Hub & Spoke Strategy. This will give us the best flexibility for most use cases. When looking at the Hub & Spoke Strategy we need to break down the requirements for the Hub cluster and the Spoke clusters.

Starting with the Hub cluster. The focus of the Hub cluster is provide enough resources for the Rancher "application" to manage the downstream clusters. The important components for the Hub cluster is cpu and memory for [etcd](https://etcd.io/). This means we need to give a little more cpu/memory head room for this cluster. Good news is that we do not need as much disk space. The number of nodes is also important. We want to ensure that we have high availability, aka redundancy. There are some good docs for [HA RKE2](https://docs.rke2.io/install/ha).

The Spoke clusters have a slightly different strategy. 



## Operating System

For this guide we are going to look at RPM based distributions for the Selinux support. Rancher Government Solutions is focused more on the security side of things. There are a few considerations. There are basically two types of nodes. Control Plane nodes are the ones that have Rancher and RKE2 management components running on them. They will need to be scaled in relation to the number applications or clusters accordingly. We will dive a little deeper when we talk about the [Rancher Deployment Strategy](#Rancher-Deployment-Strategy).

### Selinux

All nodes need Selinux enabled and in enforcing mode. This will greatly increase the defenses from anything malicious.

### Kernel Tweaks

Over the years we have seen some really good kernel tweaks for the OS when running containers. Comments are inline. All nodes should have these tweaks applied.

```bash
cat << EOF >> /etc/sysctl.conf
# SWAP settings
vm.swappiness=0
vm.panic_on_oom=0
vm.overcommit_memory=1
kernel.panic=10
kernel.panic_on_oops=1
vm.max_map_count = 262144

# Have a larger connection range available
net.ipv4.ip_local_port_range=1024 65000

# Increase max connection
net.core.somaxconn=10000

# Reuse closed sockets faster
net.ipv4.tcp_tw_reuse=1
net.ipv4.tcp_fin_timeout=15

# The maximum number of "backlogged sockets".  Default is 128.
net.core.somaxconn=4096
net.core.netdev_max_backlog=4096

# 16MB per socket - which sounds like a lot,
net.core.rmem_max=16777216
net.core.wmem_max=16777216

# Various network tunables
net.ipv4.tcp_max_syn_backlog=20480
net.ipv4.tcp_max_tw_buckets=400000
net.ipv4.tcp_no_metrics_save=1
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_syn_retries=2
net.ipv4.tcp_synack_retries=2
net.ipv4.tcp_wmem=4096 65536 16777216

# ARP cache settings for a highly loaded docker swarm
net.ipv4.neigh.default.gc_thresh1=8096
net.ipv4.neigh.default.gc_thresh2=12288
net.ipv4.neigh.default.gc_thresh3=16384

# ip_forward and tcp keepalive for iptables
net.ipv4.tcp_keepalive_time=600
net.ipv4.ip_forward=1

# monitor file system events
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576

# disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
EOF
sysctl -p
```

### CPU Memory and Storage

It makes more sense to combine CPU Memory and Storage into one section. Rancher has some good [documentation](https://ranchermanager.docs.rancher.com/pages-for-subheaders/installation-requirements#hardware-requirements) on the subject. We can be more prescriptive here.






## RKE2 - Kubernetes
https://docs.rke2.io/install/ha



## Networking




## Final Thoughts

