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

## LPTS Overview: NCS55xx and NCS5xx

LPTS module in IOS XR software classifies all ‘For Us’ control packets into 97 different flows. The incoming traffic rate limiting is done with a well-defined signature programmed in TCAM of each line-card. A signature can be anything that can identify the specific flow which the application is trying to permit. Each flow has it own hardware policer to restrict the punt traffic rate for the flow type.Today LPTS uses the following tuples to identify a packet. 
    
| Tuples              |
|---------------------|
| VRF                 |
| L3-protocol         |
| L4-protocol         |
| Interface           |
| Source Address      |
| Source Port         |
| Destination Address |
| Destination Port    |

To minimize the usage of TCAM resource, the hardware programs only the protocol default entries in hardware and punt the control packets to LC CPU for full lpts lookup. In NPU, the ‘For Us’  control packet will do a hardware TCAM lookup which will hit one of the protocol default TCAM entries. Based on lookup results, the control packets will get policed and punted to LC CPU.
When the ‘For Us’ packet is received by the LPTS decap node  in NetIO, LPTS  does ifib lookup and find the packet associated  protocol client and deliver the packet to the protocol stack.
Note that, with this approach we police all control traffic in hardware, perform full LPTS lookup in software LPTS in LC CPU before punting the packet to RP XR stack.
Post 6.2.2 release, LPTS PD supports full pifib implementation in hardware with the use of 'dynamic flow feature' where the number of LPTS TCAM entries and per FLOW max entries can be controlled via PD LPTS profile and CLI configuration. Please refer to the child page for more info on the dynamic lpts flow feature.
