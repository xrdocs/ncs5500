---
published: true
date: '2021-02-01 10:07 +0530'
title: NCS55x and NCS5xx Domain Based LPTS Policers
author: Tejas Lad
excerpt: This document will explain the domain based LPTS policers and its use cases
tags:
  - NCS5500
  - NCS500
  - LPTS
  - Control Plane Protection
  - LPTS Domain Based Filtering
position: hidden
---
{% include toc %}

## Introduction

It has been a while since we wrote the [first](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/ "first") article on LPTS. There we introduced the concept of LPTS and how it is implemented on NCS55xx and NCS5xx family of routers. We also saw with examples how LPTS entries are created in the hardware and how they can be altered as per different requirements. In this document we will explore **Domain based LPTS Policers** and understand the use case of the feature.

![Screenshot 2021-02-01 at 12.07.56 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 12.07.56 PM.png)

## Problem Definition

The usual LPTS implementation treats the traffic in a single domain and packet rate is controlled via single LPTS policer profile. LPTS policer is implemented per flow-type which are classified by LPTS. But customer needs more granular control for LPTS configuration. They require to have separate policer values for each of the ports in the router. Some LCs in ASR9K support per NPU LPTS policer profiles, where for a set of ports under 1 NPU a different policer profile can be defined. But the disadvantage is that again we do not have granularity per port. All the ports in the same NPU has to adhere the same LPTS policer defined. Therefore on NCS55xx and NCS5xx we have the provision to achieve separate policer profiles for a set of ports or domain. Domain would be logical grouping of ports. Domains will provide the capability to select LPTS policer profile independently for the defined domains. This will provide better control of ingress packets in the router.

## Advantages of using per domain LPTS 

![Screenshot 2021-02-01 at 5.28.04 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 5.28.04 PM.png)

Let us take an example of NCS 5501. It has 48 1G/10G ports and 6 40G/100G ports. The first set of ports are located in core 0 and second set in core 1. The advantage of using per domain LPTS definition are:    

  - We can use existing concept of known/default policers
  - Per port resources are available on NCS55xx and NCS5xx 
  - It strengthens the existing security plus gives added granularity
  - Domain creates individual isolation of ports (e.g VRF)
  - This can scale even with just having internal TCAM
 
Note: By default all the ports will be classified in default domain. 
{: .notice--info}

## Domain Based LPTS Architecture
 
![Screenshot 2021-02-01 at 7.49.31 PM.png]({{site.baseurl}}/images/Screenshot 2021-02-01 at 7.49.31 PM.png)
 


## Sample Use Case
 
## Memory Impact
 
## Scale
 
 The total number of domains supported would be as per platform capability.
 

 
  

