---
published: true
date: '2021-04-19 14:04 +0530'
title: BFD Architecture on NCS55xx and NCS5xx
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
{% include toc icon="table" title="BFD Architecture on NCS55xx and NCS5xx" %} 

## Introduction

After successfully introducing the [LPTS](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/) for control plane protection and the [ACL's](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/) for data-plane protection on the NCS55xx and NCS5xx portfolio, we will start this new series on faster convergence. All of us are already aware of the concept Bidirectional Forwarding Detection - BFD. BFD is fairly old technology and is widely deployed across the industry for faster network convergence. In this tech-note, we will discuss the hardware architecture and implementation of BFD on NCS55xx and NCS5xx platforms.

## Quick Refresh

![Screenshot 2021-04-19 at 2.36.38 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-19 at 2.36.38 PM.png)

Convergence of business-critical applications is a very important for any network infrastructure, whether it is service provider or any enterprise. Though today's network have various levels of redundancy, but the convergence is dependant upon the ability of individual network devices to quickly detect failures and reroute traffic to an alternate path. Bidirectional forwarding detection (BFD) provides low-overhead, short-duration detection of failures in the path between adjacent forwarding engines. BFD allows a single mechanism to be used for failure detection over any media and at any protocol layer, with a wide range of detection times and overhead. The fast detection of failures provides immediate reaction to failure in the event of a failed link or neighbor.

Another benefit of BFD, is that it provides network administrators with a consistent method of detecting failures. Thus, one availability methodology could be used, irrespective of the Interior Gateway Protocol (IGP) or the topology of the target network. This eases network profiling and planning, because re-convergence time should be consistent and predictable. BFD function is defined in [RFC 5880](https://tools.ietf.org/html/rfc5880). BFD is essentially a Control plane protocol designed to detect the forwarding path failures. BFD adjacencies do not go down on Control-Plane restarts (e.g. RSP failover) since the goal of BFD is to detect only the forwarding plane failures. [[Reference](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)] 

For understanding the BFD in details please visit the excellent articles. [[Reference1](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv4-implementation-on-ncs5500-platform/ba-p/3825926) [Reference2](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)]. In this document, we will try to focus on BFD architecture and hardware implementation.

## BFD Modes on NCS55xx and NCS5xx

| Mode         | Support |
|--------------|---------|
| Asynchronous | Yes     |
| Echo         | No      |

For further details and working on the modes of operation, please [Visit](https://www.cisco.com/c/en/us/td/docs/routers/crs/software/crs_r4-2/interfaces/configuration/guide/hc42bifw.html#wp1026021)

## BFD Single-Path and Multi-Path Sessions

BFD IPv4/IPv6 sessions over physical interfaces and sub-interfaces or bundle are referred to as Single-Path sessions.BFD sessions between virtual interfaces (BVI, BE, PWHE, GRE tunnel) of directly connected peers is also a Multi-Path session because of possible asymmetrical routing. Both BFD Single-Path and Multipath sessions are supported on NCS55xx and NCS5xx. 

### BFD Single-Path Support

| BFD Type           | v4  | v6  |
|--------------------|-----|-----|
| Physical Interface | Yes | Yes |
| Sub-Interface      | Yes | Yes |
| BFDoBundle - BoB   | Yes | Yes |

### BFD Multi-Path Support

| BFD Type                       | v4  | v6  |
|--------------------------------|-----|-----|
| BFD over Logical Bundle - BLB  | Yes | Yes |
| BVI                            | Yes | Yes |
| BGP MultiHop                   | Yes | Yes |

Note1: BFD Multi-Path (v4/v6) over BVI and BGP Multihop is not supported on systems based on J2.
{: .notice--info}

Note2: BFD Multi-Path (v6) over BVI is not supported on NCS560.
{: .notice--info}

## BFD Hardware Implementation 

![Screenshot 2021-04-20 at 12.09.21 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-20 at 12.09.21 PM.png)

All of us are already aware of the Pipeline architecture which is used for packet processing in NCS55xx and NCS5xx. Let us understand the packet processing w.r.t BFD. 

  - When a packet is received in the ingress direction, it goes through several stages in IRPP.   --   - Ingress hardware logic is capable of identifying the oam packets which is present in Forwader block in pipeline. 
  - It supports all the oam packets and the identification logic generates variables which will be used by the next Step OAM Classifier.
  - When OAMP processor receives the BFD packet.  It has 2 options
    - OAMP consume the BFD packet
    - OAMP punts the packet to CPU.
  - Upon the reception of the BFD packet, various checks are done.
  - When any of this checks fails and the corresponding sticky bit is set and destination to the CPU is picked.
  - If all the checks pass , the Packet is consumed by the OAMP machine and not punted to CPU.

### Ingress PP Pipeline

The ingress PP pipeline provides two main functions.
  - BFD identification  
  - BFD classification. 

When a BFD packet is forwarded to the OAM processor, the OAM processor must receive some information on how to process the packet. Various blocks like FLP and PMF update this OAM information in the pipeline.

### 2-Pass Processing

BFD packets are processed in 2 cycles. In first cycle, packet is recycled on a well known port for core0 and core1 before BFD packet processing starts. 

**1st cycle:**

  - BFD is treated as an IP packet and will hit the FLP entry which satisfies IP criteria and destination is resolved as recycle port.
  - The PP port value on which packet is received is extracted in PMF stage.
  - It is added as part of stacking header. 

**2nd cycle:**

  - After recycling the packet, parser will qualify the packet as BFD and would set the trap code accordingly.
  - In PMF stage, BFD packet would be processed and sets the traps & destination accordingly.




