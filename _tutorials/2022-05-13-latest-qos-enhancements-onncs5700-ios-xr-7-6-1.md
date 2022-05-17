---
published: false
date: '2022-05-13 13:49 +0530'
title: Latest QoS Enhancements onNCS5700 - IOS-XR 7.6.1
author: Paban Sarma
excerpt: >-
  This Article covers the latest QoS Innovation brought to NCS 5700 Series
  Platforms with IOS XR 7.6.1
tags:
  - iosxr
  - cisco
  - NCS 5700
---
```
Revision History:
v1: Updated as of IOS XR 7.6.1
```


## Introduction

As explained in our previous article, the queuing model on NCS 5500 is Virtual output Queues (VoQ) based and it happens on the ingress NPU in the packet path. With IOS XR 7.6.1, there is a new queuing mode intorocued on NCS 5700 system where queuing is done on the NPU where the egress port belong. This improves the overall system scale in terms of QoS scale by restricting VoQ distribution and also allows better flexibility in terms of QoS functionality. This feature is applicable to NCS 5700 system with external TCAM.

This new mode is called Egress Traffic Manager (ETM), and can be enabled on port basis while non ETM port behaves the previous way. This article will cover in depth explantion on the implementation and configuration aspects of the newly introduced ETM mode for QoS. 


## Quick Recap of VoQ Model 

## Egress Traffic Manager (ETM) Architecture and Data Path

## ETM Configuration Steps

## ETM Related Facts

## Conclusion


