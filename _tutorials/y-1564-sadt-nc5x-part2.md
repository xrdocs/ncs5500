---
published: false
date: '2021-02-03 10:16 +0530'
title: 'Y.1564 Service Activation Testing on NCS : Part2'
author: Paban Sarma
tags:
  - iosxr
  - NCS 5500
  - NCS 500
excerpt: >-
  Tutorial Explaining Y.1564 color aware mode configuration  and use case on NCS
  500 and NCS 5500 series routers
---
|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com) |  
|Chethan K .S, Software Test Lead (ches@cisco.com) |

## Overview

In [Previous Article](https://xrdocs.io/ncs5500/tutorials/y-1564-sadt-nc5x-part1/) we looked at the Y.1564 concepts and example configuration to run a Y.1564 Service Activation Test on NCS 5500 and NCS 500 routers. In this part, we will explore more capabilities such as color aware test and their application.

## Y.1564 Color Profile
An ethernet service activation test can be either color aware or color blind. By default, a test is color blind and only a single flow is generated at a rate configured in the information rate. The color aware mode is distinguished by use of a different COS marking. This different COS identifies the Excess Information Rate (EIR) flow. The committed flow uses the COS value provided in the “outer-COS” and “inner-COS” configuration. In a color aware flow, the EIR flow that is generated might have the discard eligibility indicator (DEI) set (dei =1). By default, it is not set (dei=0). The Committed information rate (CIR) needs to be configured for color-aware mode and EIR is obtained from the difference of information rate (IR) and committed information rate (CIR).

```
ethernet service-activation-test
 profile profile_#_1
  outer-cos 1
  color-aware cir 700 mbps eir-color cos 0
  information-rate 1 gbps
  packet-size 512
```

The line `color-aware cir 700 mbps eir-color cos 0`  in the SADT profile denotes that this is a color aware profile. The committed information rate (CIR) is specified in the same configuration line. The excess information rate (EIR) is obtained from the difference of information rate (IR) and the CIR . When traffic is generated, traffic within CIR is marked with the outer-cos value in profile, traffic exceeding CIR profile is marked with the eir-color cos. 

There are certain restriction on cos values that can be used in an color aware Service Activation Profile shown in the below table:
