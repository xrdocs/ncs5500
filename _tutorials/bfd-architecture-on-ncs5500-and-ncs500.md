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

As you might be already aware by now, of the Pipeline architecture which is used for packet processing in NCS55xx and NCS5xx. For details on the Pipeline Architecture please visit this excellent [article](https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-1-understanding-packet-buffering/). Let us understand the packet processing w.r.t BFD. 


### RX Path 

![Screenshot 2021-04-26 at 1.22.17 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-26 at 1.22.17 PM.png)


**Ingress PP Pipeline**

The ingress packet processing pipeline provides two main functions:
  - BFD identification. By identification it means whether it is a BFD packet (after identifying it as for-us packet).
  - BFD classification. By classification we mean a single path or a multi path BFD packet.

### 2-Pass Processing

BFD on the NCS55xx and NCS5xx is hardware offloaded. All the BFD packets are processed in 2 cycles. In first cycle, packet is recycled on a well known port (internal) for core0 and core1 before BFD packet processing starts. 

**1st cycle:**

  - L3 Lookup takes place. 
  - BFD is treated as an IP packet and check will be made whether it is a 'for-us' packet.
  - If the packet is identified as 'for-us' packet, it is recycled for second pass. 
  - The packet is then sent to the PMF stage.

**2nd cycle:**

  - After recycling the packet and the parser qualifying the packet as BFD packet, trap codes are generated. 
  - Then the packets are sent to OAMP Processor.
  - When OAMP processor receives the BFD packet.  It has 2 options
    - OAMP consume the BFD packet
    - OAMP punts the packet to CPU.

**OAMP punts the BFD packet to CPU**


-  When a BFD packet is forwarded to the OAM processor, the OAM processor must receive some information on how to process the packet. 
- Various blocks like FLP and PMF update this OAM information in the pipeline.
- Upon the reception of the BFD packet, the following checks are done.
   - Verify that the packet and the traps are correct or else generate the appropriate error code. 
   - Verify that the type taken from the trap is the correct.
   - There are various other checks which are internally done in the pipeline.
   - When any of the required checks fails and the corresponding sticky bit is set and destination to the CPU is picked
   
**OAMP consume the BFD packet**

- If all the checks pass , the Packet is consumed by the OAMP RX machine  and not punted to CPU.

Note: Highlevel definition of the blocks used in the above discussion  
OAM: Operation, administration and Management Block used for administration of the ethernet and BFD OAM  
FLP: Forwarding lookup block in the forwarding engine  
PMF: Programmable Mapping and Filtering block in the ingress and egress pipeline  
{: .notice--info}



TX Path


OAM will generate the BFD packet with complete BFD payload and UDP header 
It will also generate the partial L3 header 
Now this packet is injected into the IRPP
IRPP and ITPP will send the packet as it is to the ETPP 
In ETPP and ERPP the complete L3 header is populated and L2 header is also added to the packet
Packet is then sent out accordingly to the network interface

 
## BFD Timers

**IPv4 and IPv6**

| Type of BFD Session | Minimum Timer Supported | Default/Minimum Multipliers |
|---------------------|-------------------------|-----------------------------|
| Physical/Vlan       | 4ms                     | 3                           |
| BOB                 | 4ms                     | 3                           |
| BLB                 | 50ms                    | 3                           |
| MHOP                | 50ms                    | 3                           |


## BFD Feature Support

  - BFD is supported only in Asynchronous mode. Demand and Echo mode are not supported.
  - Static, OSPF, BGP and IS-IS applications are supported in IPv4 BFD.
  - Only static routes are supported in IPv6 BFD.
  - BFD over VRF is supported.
  - BFD over BVI is supported only on fixed NCS550 platforms.
  - BFD support over VRRP interface is supported from IOS-XR 7.2.1
  - BFD dampening for IPv4 is supported.
  - BFD multihop is supported over an IP and non-IP core. 
  - BoB and BLB coexistence is not supported at the moment. It will be supported in the future.
  - BFD with IPv4 supports upto 6 unique timers. With IPv6 we do not have limit of unique timers.
 
 Note: We will dedicate separate technotes on some of the features described above for details.
{: .notice--info}

## BFD Scale

Scale is defined by the OAM engine. 


## Reference 

  - [ASR9k BFD Implementation](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)
  - [BFD NCS5500 ](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv4-implementation-on-ncs5500-platform/ba-p/3825926)
  - [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/73x/b-routing-cg-ncs5500-73x/implementing-bfd.html)

## Thanks

Special thanks to Arun Vadamalai for his valueable feedback during the documentation.

## Summary

This was an introductory article on BFD to get familiarise with feature wi
th pipeline architecture. In the next article, we will focus on how to read the BFD parameters in the outputs and also see some basic debugging. Also we will touch upon the concept of BOB and BLB.
