---
published: true
date: '2021-09-28 15:34 +0530'
title: Egress IPv6  ACLs on NCS5500
author: Tejas Lad
excerpt: >-
  This document will describe IPv6 ACL Egress ACLs on NCS5500 J/J+ and highlight
  the enhancements when it comes to J2
tags:
  - iosxr
  - NCS5500
  - ncs5500
  - J2
  - Jericho2
  - ACL
  - IPv6
  - Egress ACL
position: hidden
---
{% include toc icon="table" title="Egress IPv6  ACLs on NCS5500" %} 

## Introduction

In the [previous article](https://xrdocs.io/ncs5500/tutorials/acl-s-on-ncs5500-bvi-interfaces/), we discussed the ACLs on NCS5500 BVI interfaces. We covered the ACL implementation and support for IPv4 and IPv6 ACLs in both ingress and egress directions. We also discussed the enhancements w.r.t IPv6 Egress ACLs. In this article we discuss the overall implementation of IPv6 Egress ACLs across NCS5500 product family (J/J+ and J2).

## Overview

![Screenshot 2021-09-29 at 2.16.58 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 2.16.58 PM.png)

Let us start with the earlier generations of NCS5500. As we already know, platforms based on J/J+ have limited hardware resources. They have very limited capabilities in the Egress PMF as well. PMF means Programmable Mapping and Filtering block in the ingress and egress pipeline. It is the most programmable and the last programmable block in the pipeline. It has all the history of the packet from other blocks (incoming port, lookup results, etc). We can override here every decision taken along the pipeline. Here we do ACL, QoS, LPTS classification and set actions (counters, policers, Traffic Class).  Egress PMF is capable of doing internal TCAM lookup for egress ACLs

![Screenshot 2021-09-29 at 2.36.05 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 2.36.05 PM.png)

In order to support the match criteria that is needed for egress IPv6 ACL, resources in the Ingress PMF block will be used. Therefore, this requires us to recycle the IPv6 packets so that the ACL can be performed by the Ingress PMF. 


