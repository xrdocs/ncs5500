---
published: true
date: '2020-09-08 14:36 +0530'
title: ACL Based Forwarding and Object Tracking for NCS5xx and NCS55xx
author: Tejas Lad
excerpt: >-
  This document aims to cover the concept of ABF and the used of object tracking
  with ABF
tags:
  - NCS5500
  - NCS500
  - NCS5xx
  - NCS55xx
  - ACL
  - ABF
  - ABF_OT
position: hidden
---
{% include toc %}

## Introduction

In today’s converged networks, operators have to implement packet forwarding in a way that goes beyond traditional routing protocol. Sometimes operators want certain traffic to be engineered based on rules which takes a separate path then what is already computed by the dynamic routing protocols. ACL Based Forwarding (ABF) can be used as a technique to achieve the same. ABF is a subset of PBR (Policy Based Routing) infrastructure in the XR platform. ABF allows traffic matching specific ACL rule to be forwarded to user specified nexthop instead of route selected by a routing protocol. ABF does not modify anything in the packet itself and therefore only affects the local routing/forwarding decision. The path for reaching the ABF next-hop, however, is determined by the normal routing/forwarding table.

![Screenshot 2020-09-08 at 3.23.09 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 3.23.09 PM.png)

## Different Types of ABF

There are 3 types of ABF supported on NCS55xx and NCS5xx

| ABF Type                                | Details                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ABF for IPv4/IPv6                       | Only the next-hop IP address(es) is specified in the ACE rule.   The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE rule,  instead of the next-hop determined by normal routing/forwarding.   The default VRF is used for the ABF next-hop lookup (determining the path) regardless of the VRF on which the  packet was received.                                     |
| VRF-aware ABF for IPv4/IPv6             | Both the next-hop IP address(es) and the next-hop VRF(s) are specified in the ACE rule.   The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE rule,  instead of the next-hop determined by normal routing/forwarding.   The specified VRF (instead of the VRF associated with the ingress packet/interface) is used for the ABF next-hop  lookup (determing the path). |
| VRF-select (VRF only) ABF for IPv4/IPv6 | Only the next-hop VRF(s) is specified in the ACE rule.  The matching traffic is forwarded to the next-hop as  determined by a destination address lookup performed in the first "up" VRF as specified in the  ACE rule (instead of the VRF associated with the ingress packet/interface).                                                                                                                |



## Reference

  - Reference 1: https://community.cisco.com/t5/service-providers-documents/asr9000-xr-abf-acl-based-forwarding/ta-p/3153403
  - 


