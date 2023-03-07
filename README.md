---
title: Reference Architecture for the Rancher Stack
author: Zack Brady - Field Engineer, Andy Clemenko - Field Engineer
contact: zack.brady@rancherfederal.com, andy.clemenko@rancherfederal.com
---

# Reference Architecture for the Rancher Stack

![rgs-banner](./images/logo_long.jpg)

## Table of Contents

* [About Us](#about-us)
* [Introduction](#introduction)
* [Rancher Stack](#rancher-stack)
* [Rancher Deployment Strategy](#rancher-deployment-strategy)
* [Node Sizing | CPU, Memory, and Storage](#node-sizing--cpu-memory-and-storage)
  * [Operating System](#operating-system)
  * [Selinux](#selinux)
  * [Kernel Tweaks](#kernel-tweaks)
  * [RKE2 - Kubernetes](#rke2---kubernetes)
* [Networking](#networking)
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

![spoke](./images/topo.jpg)

The good news is that there is quite a bit of [documentation](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/rancher-deployment-strategy) on the two Rancher Deployment Strategies. We are going to focus on the Hub & Spoke Strategy. This will give us the best flexibility for most use cases. When looking at the Hub & Spoke Strategy we need to break down the requirements for the Hub cluster and the Spoke clusters.

Starting with the Hub cluster. The focus of the Hub cluster is provide enough resources for the Rancher "application" to manage the downstream clusters. The important components for the Hub cluster is cpu and memory for [etcd](https://etcd.io/). This means we need to give a little more cpu/memory head room for this cluster. Good news is that we do not need as much disk space. The number of nodes is also important. We want to ensure that we have high availability, aka redundancy. There are some good docs for [HA RKE2](https://docs.rke2.io/install/ha). The Rancher [documentation](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/tips-for-running-rancher) has a few tips to think about.

The Spoke clusters have a slightly different strategy. [HA](https://docs.rke2.io/install/ha) is still important. The main difference is around storage and the node sizing to accommodate the anticipated application load. Meaning ensure that there are enough nodes with resources to handle the level of failure the applications and handle. This can be tricky to figure out. Best rule of thumb is have around 30% of the cluster's total resources free. Feel free to reach out to us if you want to deep dive on capacity planning. The Rancher docs has a small section on [capacity planning](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/tips-for-running-rancher#monitor-your-clusters-to-plan-capacity) that is worth a read.

Remember we do not have to create a cluster with heterogeneous nodes. We can vary the nodes sizes and counts to fit our specific needs.

## Node Sizing | CPU, Memory, and Storage

As we just highlighted we have two different strategies for the Hub and Spoke clusters. There is another differentiator to think about. Control Plane nodes versus Worker nodes. Control Plane nodes are used for the management of the local cluster with components like the API server and etcd. The Workers are where the applications are run from. Speaking of applications, it makes more sense to combine CPU Memory and Storage into this section. Rancher has some good [documentation](https://ranchermanager.docs.rancher.com/pages-for-subheaders/installation-requirements#hardware-requirements) on the subject. We can be more prescriptive here. Keep in mind that you can run other "Enterprise" applications on the Hub cluster. Several that come to mind are [Harbor (Registry)](https://goharbor.io/) and [Gitea (Version Control)](https://gitea.io/en-us/).

| Cluster Type   | CPU | Memory | Disk | Number of Nodes |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| Hub - Control Plane |  4 Core | 8 GB | 100 GB| 3 |
| Hub - Worker |  8 Core | 16 GB | 200 GB | 3+ |
| Spoke   |  4 Core | 8 GB| 100 GB | 3 |
| Spoke   |  8 Core | 16 GB | 200 GB| 3+ |

Please note that the values above are starting points. If your applications are not that disk intensive, or you plan on using cloud provided storage you can lower the numbers. All nodes should leverage SSD or faster storage. All nodes should have their load averages monitored. This will give you an indication of when the nodes needs to be scaled. Control Plane nodes should be scaled with more cpu and memory. The worker nodes should be scaled with more nodes. Observability is going to really important here. Good thing Rancher has Grafana, Prometheus, Fluentd charts built in.

One strategy that we might want to consider is the "lot of little" nodes. This strategy works handles failures and upgrades really well. Meaning when a node goes down there is not a lot of the total capacity lost. Case in point, if we have 4 worker nodes and one goes down we have lost 1/4 or 25% of the total capacity. Where as with 10 worker nodes, we would only lose 10% of the total capacity.

### Operating System

For this guide we are going to look at RPM based distributions for the Selinux support. Rancher Government Solutions is focused more on the security side of things. Rocky/RHEL are the preferred Operation Systems. Debian based ones will work as well.

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

Now we can look at Kubernetes.

## RKE2 - Kubernetes

Of course we prefer [RKE2](https://docs.rke.io) for the simple fact that is has a DISA STIG. There is a nice article about it from [Businesswire](https://www.businesswire.com/news/home/20221101005546/en/DISA-Validates-Rancher-Government-Solutions%E2%80%99-Kubernetes-Distribution-RKE2-Security-Technical-Implementation-Guide). We even has a [tl;dr](https://github.com/clemenko/rke2/blob/main/RKE2_STIG.md) with the config. One important thing to implement is that all the Control Plane nodes should be configured in [High Availability](https://docs.rke2.io/install/ha) mode. At a high level the three Control Plane nodes should use DNS round robin or a Layer 4 Load Balancer. The [docs](https://docs.rke2.io/install/ha) go into greater detail. And the other good news is that RKE2 is 100% air gap compliant.

Bottom Line:

* Control Plane nodes in High Availability
* All nodes STIG'd

Let's take a look at how all the nodes are connected.

## Networking

![layout](./images/layout.jpg)

Networking can be painful if we let it be. If we break the problem down it can be easy. Let's start with the official [Port List](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements/port-requirements). We can see that there are potentially a lot of ports that need to be open. For this reason I recommend to keep the firewalls and security groups wide open between the nodes. What helps with this is if each cluster is contained within a subnet or VPC. This makes the security groups and firewalls easier to manage. Another major consideration is around the Rancher Load Balancer and Ingress. Ideally we want to have at least two dedicated Load Balancer for Ingress. Each cluster will have an external Load Balancer pointing to the Ingress ports.

## Final Thoughts

