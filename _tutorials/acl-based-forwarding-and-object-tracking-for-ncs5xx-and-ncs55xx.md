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

There are 3 types of ABF supported on NCS55xx and NCS5xx [Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/72x/b-ip-addresses-cg-ncs5500-72x/m-implementing-access-lists-prefix-lists-ncs5500.html "Reference")

  - **ABF for IPv4/IPv6**
  
    - Only the next-hop IP address is specified in the ACE rule.
	- The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - Default VRF is used in this type of ABF.
    
    ![Screenshot 2020-09-08 at 4.21.12 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.12 PM.png)


  - **VRF-aware ABF for IPv4/IPv6**
  
    - Both the next-hop IP address and the next-hop VRF are specified in the ACE rule.
    - The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - The specified VRF is used instead of the default VRF for determining the path.
    
    ![Screenshot 2020-09-08 at 4.21.19 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.19 PM.png)


  - **VRF-select ABF for IPv4/IPv6** 
  
    - Only the next-hop VRF(s) is specified in the ACE rule.
    - The matching traffic is forwarded to the first “UP” VRF as specified in the ACE rule.
    - Supported from IOS-XR 6.5.1 onwards.
    
    ![Screenshot 2020-09-08 at 4.21.26 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.26 PM.png)




## Reference

  - Reference 1: https://community.cisco.com/t5/service-providers-documents/asr9000-xr-abf-acl-based-forwarding/ta-p/3153403
  - 


