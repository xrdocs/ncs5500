---
published: true
date: '2022-04-18 13:10 +0530'
title: Latest ACL Enhancements on NCS5500 - IOS-XR 7.6.1
author: Tejas Lad
excerpt: >-
  This document will introduce the behaviour new enhancements w.r.t ACL feature
  on NCS5500
tags:
  - iosxr
  - cisco
  - ACL
  - NCS5500
  - access-list
  - access control list
  - data plane
position: hidden
---
{% include toc icon="table" title="Latest ACL Enhancements on NCS5500 - IOS-XR 7.6.1" %} 
## Introduction

In our [previous article](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/), we discussed the ACL enhancements on the newer generation NCS5500 plaforms. In this article we will discuss some more enhancements w.r.t ACLs, which we have done in IOS-XR 7.6.1

## IOS-XR 7.6.1 Enhancements

In IOS-XR 7.6.1, we have brought in a number of new enhancements when it comes to ACLs. 
 - Increased Ingress ACLs
 - ACL Chaining with ACL Based Forwarding (ABF)
 - Enable Ingress Interface Logging on ACE
 - ACL-Based Policing

We will discuss about each enhancement in details

## Increased Ingress ACLs


In earlier releases, we could configure maximum upto 127 different traditional ingress ACLs and 255 different hybrid ingress ACLs in shared ACL mode per line card. From IOS-XR 7.6.1 we can now configure an increased number of either traditional (non-compression) or hybrid (compression) ingress ACLs in shared ACL mode, as listed below:

 - A maximum of 512 different traditional ingress ACLs per line card.
 - A maximum of 1000 different hybrid ingress ACLs per line card.

Increased ACLs provide you with enhanced traffic filtering capabilities to control how traffic packets move through the network and restrict the access of users and devices to the network. For further deepdive please follow the [link](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/76x/b-ip-addresses-cg-ncs5500-76x/m-implementing-access-lists-prefix-lists-ncs5500.html#id_63255). 

## ACL Chaining with ACL Based Forwarding (ABF)

Prior to IOS-XR 7.6.1, ABF and ACL chaining with Common ACL were mutually exclusive features. From 7.6.1 onwards, we can enable ABF in conjunction with ACL chaining in Common ACL. With this feature, the router can inspect and forward the packets based on the ABF rule in Common ACL. In earlier releases. For further deepdive, please follow the links for [Chained ACL](https://xrdocs.io/ncs5500/tutorials/chained-acl-for-ncs55xx-and-ncs5xx/) and [ABF](https://xrdocs.io/ncs5500/tutorials/acl-based-forwarding-and-object-tracking-for-ncs5xx-and-ncs55xx/). 




