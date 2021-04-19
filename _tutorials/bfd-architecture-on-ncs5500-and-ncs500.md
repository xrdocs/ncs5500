---
published: true
date: '2021-04-19 14:04 +0530'
title: BFD architecture on NCS5500 and NCS500
author: Tejas Lad
excerpt: Implementation of BFD on NCS5500 and NCS500
tags:
  - NCS5500
  - NSC500
  - NCS540
  - NCS 5500
  - BFD
  - Protection
position: top
---
## Introduction

After our successfully introducing the [LPTS](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/) for control plane protection and the [ACL's](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/) for data-plane protection, we will start this new series on faster convergence. All of us are already aware of the concept Bidirectional Forwarding Detection - BFD. We use BFD for faster network convergence. In this tech-note, we will discuss the hardware architecture and implementation of BFD on NCS5500 and NCS500 platforms.

## Quick Refresh

![Screenshot 2021-04-19 at 2.36.38 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-19 at 2.36.38 PM.png)

Convergence of business-critical applications is a very important for any network infrastructure, whether it is service provider or any enterprise. Though today's network have various levels of redundancy, but the convergence is dependant upon the ability of individual network devices to quickly detect failures and reroute traffic to an alternate path. Bidirectional forwarding detection (BFD) provides low-overhead, short-duration detection of failures in the path between adjacent forwarding engines. BFD allows a single mechanism to be used for failure detection over any media and at any protocol layer, with a wide range of detection times and overhead. The fast detection of failures provides immediate reaction to failure in the event of a failed link or neighbor.