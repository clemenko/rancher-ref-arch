---
title: Production Reference Architecture for the Rancher Stack
author: Zack Brady - Field Engineer, Andy Clemenko - Field Engineer
contact: zack.brady@rancherfederal.com, andy.clemenko@rancherfederal.com
---

![rgs-aws-banner](/images/rgs-banner-rounded.png)

# Production Reference Architecture for the Rancher Stack

### Table of Contents

* [About Us](#about-us)
* [Introduction](#introduction)
* [Rancher Deployment Strategy](#Rancher-Deployment-Strategy)

* [Final Thoughts](#final-thoughts)

## About Us

A little bit Zack Brady, his history, and what he's done in the industry.

* DOD/IC Contractor
* U.S. Military Veteran
* Open-Source Contributor
* Built and Exited a Digital Firm
* Active Volunteer Firefighter/EMT

A little bit Andy Clemenko, his history, and what he's done in the industry.

* DOD/IC Contractor
* Docker Federal (PS, SE, TAM)
* RedHat/Stackrox Federal (SE)
* Rancher Advocate (6+ Years)
* Volunteer Firefighter/EMT

## Introduction

It's almost impossible to configure and implement a solution to fit every customer, every environment, and every situation. Hopefully this reference architecture will be able to provide you a great starting point to apply to your customer, your environment, and your situation. This guide is intended to highlight an idea deployment of Rancher and a few downstream clusters.

We are going to look at the Rancher stack in a slightly different order. This is due to Rancher being the primary decision driver.

## Rancher Deployment Strategy

The good news is that there is quite a bit of [documentation](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/rancher-deployment-strategy) on the two Rancher Deployment Strategies. We are going to focus on the Hub & Spoke Strategy. This will give us the best flexibility for most use cases. When looking at the Hub & Spoke Strategy we need to break down the requirements for the Hub cluster and the Spoke clusters. 





## Operating Systems - Rocky

## RKE2 - Kubernetes
https://docs.rke2.io/install/ha






## Final Thoughts

