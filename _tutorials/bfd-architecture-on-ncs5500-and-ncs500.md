---
published: true
date: '2021-04-19 14:04 +0530'
title: BFD architecture on NCS55xx and NCS5xx
author: Tejas Lad
excerpt: Implementation of BFD on NCS5500 and NCS500
tags:
  - NCS5500
  - NSC500
  - NCS540
  - NCS 5500
  - BFD
  - Protection
position: hidden
---
## Introduction

After successfully introducing the [LPTS](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/) for control plane protection and the [ACL's](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/) for data-plane protection on the NCS55xx and NCS5xx portfolio, we will start this new series on faster convergence. All of us are already aware of the concept Bidirectional Forwarding Detection - BFD. BFD is fairly old technology and is widely deployed across the industry for faster network convergence. In this tech-note, we will discuss the hardware architecture and implementation of BFD on NCS55xx and NCS5xx platforms.

## Quick Refresh

![Screenshot 2021-04-19 at 2.36.38 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-19 at 2.36.38 PM.png)

Convergence of business-critical applications is a very important for any network infrastructure, whether it is service provider or any enterprise. Though today's network have various levels of redundancy, but the convergence is dependant upon the ability of individual network devices to quickly detect failures and reroute traffic to an alternate path. Bidirectional forwarding detection (BFD) provides low-overhead, short-duration detection of failures in the path between adjacent forwarding engines. BFD allows a single mechanism to be used for failure detection over any media and at any protocol layer, with a wide range of detection times and overhead. The fast detection of failures provides immediate reaction to failure in the event of a failed link or neighbor.

Another benefit of BFD, is that it provides network administrators with a consistent method of detecting failures. Thus, one availability methodology could be used, irrespective of the Interior Gateway Protocol (IGP) or the topology of the target network. This eases network profiling and planning, because re-convergence time should be consistent and predictable. BFD function is defined in [RFC 5880](https://tools.ietf.org/html/rfc5880). BFD is essentially a Control plane protocol designed to detect the forwarding path failures. BFD adjacencies do not go down on Control-Plane restarts (e.g. RSP failover) since the goal of BFD is to detect only the forwarding plane failures. [[Reference](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)] 

For understanding the BFD in details please visit the excellent articles. [[Reference1](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv4-implementation-on-ncs5500-platform/ba-p/3825926) [Reference2](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)]. In this document, we will try to focus on BFD architecture and hardware implementation.

## BFD on NCS55xx and NCS5xx

