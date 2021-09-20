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
position: top
---
{% include toc icon="table" title="BFD Architecture on NCS55xx and NCS5xx" %} 

## Introduction

After successfully introducing the [LPTS](https://xrdocs.io/ncs5500/tutorials/introduction-to-ncs55xx-and-ncs5xx-lpts/) for control plane protection and the [ACL's](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/) for data-plane protection on the NCS55xx and NCS5xx portfolio, we will start this new series on faster convergence. All of us are already aware of the concept Bidirectional Forwarding Detection - BFD. BFD is fairly old technology and is widely deployed across the industry for faster network convergence. In this tech-note, we will discuss the implementation of BFD w.r.t NCS55xx and NCS5xx platforms.

## Quick Refresh

![Screenshot 2021-04-19 at 2.36.38 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-19 at 2.36.38 PM.png)

Convergence of business-critical applications is a very important for any network infrastructure, whether it is service provider or any enterprise. Though today's network have various levels of redundancy, but the convergence is dependant upon the ability of individual network devices to quickly detect failures and reroute traffic to an alternate path. Bidirectional forwarding detection (BFD) provides low-overhead, short-duration detection of failures in the path between adjacent forwarding engines. BFD allows a single mechanism to be used for failure detection over any media and at any protocol layer, with a wide range of detection times and overhead. The fast detection of failures provides immediate reaction to failure in the event of a failed link or neighbor.

Another benefit of BFD, is that it provides network administrators with a consistent method of detecting failures. Thus, one availability methodology could be used, irrespective of the Interior Gateway Protocol (IGP) or the topology of the target network. This eases network profiling and planning, because re-convergence time should be consistent and predictable. BFD function is defined in [RFC 5880](https://tools.ietf.org/html/rfc5880). BFD is essentially a Control plane protocol designed to detect the forwarding path failures. BFD adjacencies do not go down on Control-Plane restarts (e.g. RSP failover) since the goal of BFD is to detect only the forwarding plane failures. [[Reference](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)] 

For understanding the BFD in details please visit the excellent articles. [[Reference1](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv4-implementation-on-ncs5500-platform/ba-p/3825926) [Reference2](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)]. In this document, we will try to focus on BFD architecture and its hardware implementation on NCS55xx and NCS5xx product families.

## BFD Modes Supported on NCS55xx and NCS5xx

| Mode         | Support |
|--------------|---------|
| Asynchronous | Yes     |
| Echo         | No      |

For further details and working on the modes of operation, please [Visit](https://www.cisco.com/c/en/us/td/docs/routers/crs/software/crs_r4-2/interfaces/configuration/guide/hc42bifw.html#wp1026021)

## BFD Single-Path and Multi-Path Sessions

BFD IPv4/IPv6 sessions over physical interfaces and sub-interfaces or bundle are referred to as Single-Path sessions. BFD sessions between virtual interfaces (BVI, BE, GRE tunnel) of directly connected peers is referred to as Multi-Path session because of possible asymmetrical routing. Both BFD Single-Path and Multipath sessions are supported on NCS55xx and NCS5xx. 

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

Note1: BFD Multi-Path (v4/v6) over BVI and BGP Multihop is supported on systems based on J2 from IOS-XR 7.4.1 in native and compatible mode.
{: .notice--info}

Note2: BFD Multi-Path (v6) over BVI is not supported on NCS560. It will be supported in future releases.
{: .notice--info}

## BFD Hardware Implementation 

BFD on the NCS55xx and NCS5xx is hardware offloaded. The hardware offload feature enables the offload of a BFD session to the network processing unit - NPU of the line cards on modular chassis or a standalone fixed chassis, in an IPv4 or IPv6 network. BFD hardware offload improves scale and reduces the overall network convergence time by sending rapid failure detection packets to the routing protocols for recalculating the routing table. NCS55xx and NCS5xx uses Pipeline architecture for packet processing. For details on the Pipeline Architecture please visit this excellent [article](https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-1-understanding-packet-buffering/). For further details on BFD hardware offload please [visit](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/66x/b-routing-cg-ncs5500-66x/b-routing-cg-ncs5500-66x_chapter_0111.html#id_103405). Let us understand the packet processing in the pipeline architecture w.r.t BFD.


### RX Path 

![Screenshot 2021-04-26 at 1.22.17 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-26 at 1.22.17 PM.png)


**Ingress PP Pipeline**

The ingress packet processing pipeline provides two main functions:
  - BFD identification: Whether it is a BFD packet (after identifying it as for-us packet).
  - BFD classification: Whether single path or a multi path BFD packet.

**2-Pass Processing**

 All the BFD packets are processed in 2 cycles. In first cycle, packet is recycled on a well known port (internal) for core0 and core1 before BFD packet processing starts. 

**1st cycle:**

  - The first pass is based on L3 Lookup. 
  - BFD is treated as an IP packet and check will be made whether it is a 'for-us' packet.
  - If the packet is identified as 'for-us' packet, it is recycled for second pass. 
  - The packet is then sent to the PMF stage.
  - If the packet is not a for-us packet then based on the L3 lookup appropriate action is taken on the packet.

**2nd cycle:**

  - After recycling the packet, parser qualifies the packet as BFD with parser code. 
  - Parser is the block which extracts ethertype, MAC addresses and determines offset (where network header starts) for next stages in the pipeline
  - There is a specific parser code for BFD packet which is internal to the hardware. 
  - BFD FLP is hit based on the parser context and trap code is generated accordingly.
  - In PMF stage, BFD packet is processed and sets the required traps & destination accordingly. 
  - The packets are then sent to OAMP Engine.
  - When OAMP Engine receives the BFD packet, it has 2 options:
    - OAMP Engine consumes the BFD packet
    - OAMP Engine punts the packet to CPU.

**OAMP Engine punts the BFD packet to CPU**


-  When a BFD packet is forwarded to the OAMP engine, it must receive some information on how to process the packet. 
- Various blocks like FLP and PMF update this information in the pipeline and pass it on to the OAM engine.
- Upon the reception of the BFD packet, the following checks are done.
   - Verify that the packet and the traps are correct or else generate the appropriate error code. 
   - Verify that the type taken from the trap is the correct.
   - There are various other checks which are internally done in the pipeline.
   - When any of the required checks fails, the corresponding failure bit is set and destination to the CPU is picked.
   - The packet is then punted to the CPU.
   
**OAMP Engine consumes the BFD packet**

- If all the checks pass without any failure bit set, the packet is consumed by the OAMP engine and processed. It is not punted to CPU.

**Highlevel definition of the blocks used in the above discussion**

| Block | Description                                                                                                                                                                                                                                                                                                                                                                  |
|-------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| OAM   | Operation, administration and Management used for administration of the ethernet and BFD OAM                                                                                                                                                                                                                                                                                 |
| OAMP Engine | It is a dedicated hardware engine to accelerate the handling of OAM packets. Main functionalities include:<br>-Generates/Receives BFD packets<br>-Generates timeout events for a session if continuous packets are not received.<br>-Punts packet to LC-CPU (BFD stack) in case there is change in BFD session state or flags. Based on which state machine is maintained. |
| FLP   | Forwarding lookup block in the forwarding engine in the IRPP.<br>-It is a very programmable and flexible block.<br>-It helps in lookup action using different database like LEM, LPM etc.<br>-It has place for OAM classification too.                                                                                                                                       |
| PMF   | Programmable Mapping and Filtering block is another block present in the pipeline.<br>-It is most programmable block in the pipeline. <br>-It has all the history of the packet from other blocks like incoming ports, lookup results etc.<br>-It takes care of ACL, LPTS, QoS etc. It is there in ingress and egress pipeline.                                              |

Note:
There are multiple blocks in the IRPP and ERPP in the pipeline for packet processing. Parser, FLP, PMF etc are one of those. This is specific to ASIC used, hence not exposing the internal details.
{: .notice--info}
### TX Path

![Screenshot 2021-04-26 at 4.30.35 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-26 at 4.30.35 PM.png)

- OAMP Engine will generate the BFD packet with complete BFD payload, L3 partial header and UDP header 
- This packet is then injected into the IRPP for further processing.
- IRPP and ITPP will send the packet to the ETPP. 
- In ETPP and ERPP the complete L3 header is populated and L2 header is also added to the packet
- Packet is then sent out accordingly from the network interface.

Note:
OAMP Engine has various blocks which are internal and cannot be published. We have used a single block for simplication
{: .notice--info}

## BFD Feature Support

  - BFD is supported only in Asynchronous mode. Demand and Echo mode are not supported.
  - Static, OSPF, BGP and IS-IS applications are supported in IPv4 BFD.
  - Only static routes are supported in IPv6 BFD.
  - BFD over VRF is supported.
  - BFD over BVI is supported only on fixed NCS5500 platforms.
  - BFD support over VRRP interface is supported from IOS-XR 7.2.1
  - BFD dampening for IPv4 is supported.
  - BFD multihop is supported over an IP and non-IP core. 
  - BoB and BLB coexistence is not supported at the moment. It will be supported in the future.
  - BFD with IPv4 supports upto 6 unique timers. With IPv6 we do not have limit of unique timers.
 
 Note: We will dedicate separate technotes on some of the features described above for details.
{: .notice--info}


## BFD Scale

Scale on NCS55xx and NCS5xx is always a subject of discussion because of the hardware resources available on the chipsets. The hardware resources are limited and has to be utilised properly to achieve the best results. Let us discuss in more details on how BFD scale is determined for these platforms and how well we have managed to address the available resources. 

Scale is decided on multiple things. First is obviously the hardware resources. From BFD feature perspective, the OAMP Engine resources play a critical role. The resources are equally divided between CFM and BFD feature needs. Now if we double-click on BFD, then within that resources are again divided for BFD Single-Path and BFD Multi-Path. Again one more consideration, we need to divide the resources amongst IPv4 and IPv6. For IPv4 we only need to worry from the OAMP resources. But for IPv6 we also need to take into account other internal resources which limits the scale (internal to ASIC). IPv6 needs almost three times more resources than IPv4 w.r.t OAMP engine. Again when the packets are recycled, we need to take into consideration the queue and its bandwidth shared with other features and number of recycles. All these factors goes into scale considerations. Each Asic type i.e. Q-MX/J/J+/J2 has different processing, resources and bandwidth capacity. So the scale will vary across chipsets. 

Considering all the above criterias, we have succeeded in carving and the resources optimally , to provide right amount of BFD sessions (v4/v6) which suits every use cases. 

Note:
If you are looking for details on the scale per product please contact us or your cisco representative.
{: .notice--info}

## BFD Timers 

NCS55xx and NCS5xx BFD Implementation supports timers which can be used for faster detection of failures. These are configurable values and users have the flexibility of configuring different timers and multiplier values. BFD v4 sessions do not have any limitations in scale w.r.t minimum timer values. But the BFD v6 scale limit does depend on the configured minimum timer. Below is the list of timers and multiplier values supported.


**Support for IPv4 Timers and Multipliers**

| Type of BFD Session | Minimum Timer Supported | Default/Minimum Multipliers |
|---------------------|-------------------------|-----------------------------|
| Physical/Vlan       | 4ms                     | 3                           |
| BOB                 | 4ms                     | 3                           |
| BLB                 | 50ms                    | 3                           |
| Multi-HOP           | 50ms                    | 3                           |

Note:
Only 6 unique timers are supported with BFDv4.  
Scale numbers for v4 will not be affected by the timer values
{: .notice--info}

**Support for IPv6 Timers and Multipliers**

| Type of BFD                 | Minimum Timer Supported         | Default/Minimum Multipliers | Scale Limitation w.r.t min_interval |
|-----------------------------|---------------------------------|-----------------------------|-------------------------------------|
| Single Hop                  | 1. 4/5/6/7 ms<br>2. 8ms & above | 3ms                         | 1. 150<br>2. 250                    |
| BOB (BFD o/ Bundle Members) | 1. 4/5/6/7 ms<br>2. 8ms & above | 3ms                         | 1. 150<br>2. 250                    |
| BLB (BFD o/ Logical bundle) | 50 ms                           | 3ms                         | 250                                 |
| MHOP                        | 50 ms                           | 3ms                         | 250                                 |

Note:
No restrictions on number of unique timers support with BFDv6.  
Scale numbers for v6 will be affected by the timer values. The above values are for J/J+/J2 compatible mode. For J2 native mode this numbers will be a bit higher. For other chipsets the numbers will vary. For detailed scale numbers please get in touch with us or your cisco representative.
{: .notice--info}

## Reference 

  - [ASR9k BFD Implementation](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)
  - [BFD NCS5500 ](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv4-implementation-on-ncs5500-platform/ba-p/3825926)
  - [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/73x/b-routing-cg-ncs5500-73x/implementing-bfd.html)
  - [NCS5500 Deepdive Cisco Live](https://www.ciscolive.com/global/on-demand-library.html?search=nicolas%20fevrier#/session/1564610723886001c86Q)

## Thanks

Special thanks to Arun Vadamalai for his valueable feedback during the documentation.

## Summary

This was an introductory article on BFD to get familiarise with feature with pipeline architecture. In the next article, we will focus more on the technical aspects of how to read the BFD parameters in the outputs and also see some basic debugging. We will touch upon the OAMP engine resource usage. We will discuss the concepts of BLB and BoB and look at its difference and use cases. Stay tuned !!!!
