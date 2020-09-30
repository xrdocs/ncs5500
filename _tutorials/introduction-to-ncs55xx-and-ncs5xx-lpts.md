---
published: true
date: '2020-09-30 16:02 +0530'
title: Introduction to NCS55xx and NCS5xx LPTS
author: Tejas Lad
excerpt: Introduction to NCS55xx and NCS5xx LPTS
tags:
  - NCS5500
  - NCS500
  - CoPP
  - control plane
  - LPTS
position: hidden
---
{% include toc %}

## Introduction

![Screenshot 2020-09-30 at 4.49.49 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-30 at 4.49.49 PM.png)

In our previous tech-notes, we focussed on exploring the data-plane security capabilities of NCS5xx and NCS55xx. In continuation to the same, we will discuss the NCS55xx and NCS5xx capabilities around control plane protection. In today's networks, control plane security is as important as data plane security. Customers need to focus on protecting the business critical control and management planes from malicious attacks and also misconfiguration events which may overwhelm the processor and bring down the router resulting in unfortunate event for any customer. The Cisco IOS devices have the concept of control plane policing. In IOS-XR devices, we use a very comprehensive and powerful feature called Local Packet Transport Services. ([Reference](https://community.cisco.com/t5/service-providers-documents/asr9000-xr-local-packet-transport-services-lpts-copp/ta-p/3123792 "Reference"))

In IOS-XR LPTS, as part of **for-us** packet delivery process, the rate at which packets are delivered are selectively monitored to avoid overwhelming the CPU. LPTS filters and police the packets based on the defined flow-type rate in hardware before punting to the software. The LPTS  takes the Cisco IOS CoPP concept to a new level by automatically applying rate limiting to all packets that must be handled by any route processor on the device. LPTS maintains tables describing all packet flows destined for the route processor and uses a set of predefined policers that are applied based on the flow type of the incoming control traffic. The use case of this is we achieve automated control to network health without relying on network operators as it becomes  difficult to configure these functions manually in large-scale networks.

## 

