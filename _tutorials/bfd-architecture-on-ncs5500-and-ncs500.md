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



OAMP processor receives the BFD packet.  It has 2 options

OAMP consume the BFD packet
OAMP punts the packet to CPU.OAMP punts the packet to CPU.

Upon the reception of the BFD packet, the following checks are done.

1. Verify that the packet is correct - In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_ERR_PACKET will be set

2. Verify that the trap is correct - In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_TRAP_PACKET will be set

3. Verify that the type (taken from the trap) is the same as that on OAMP_MEP_DB.TYPE In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_TYPE_MISS will be set.

4. Find the RMEP DB Entry. This is done with an exact match table in the OAMP (EMC).

   For BFD, input of the EMC table is [PROJECT:OAM-ID, Cpu-Configurable value]. Output of the EMC table is the   OAMP_RMEP_DB index .In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_EMC_MISS will be set

5. Verify packet packet.Your-Discriminator vs. MEP’s My Discriminator (This is taken from the MEP_DB index + OAMP_BFD_MY_DISCRIMINATOR_RANGE_START).In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_MY_DISC_MISS will be set

6. Verify packet packet.My-Discriminator vs. MEP’s Your-Discriminator (The latter taken directly from the MEP-DB entry). In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_YOUR_DISC_MISS will be set.

7. Verify packet SIP vs. that MEP_DB’s DIP (OAMP_MEP_DB_BFD_ON_IPV4_MULTI_HOP.DST_IP). In case of failure OAMP_VALIDITY_CHECK_STICKY. STICKY_PORT_SRC_IP_MISS will be set.

When any of this checks fails and the corresponding sticky bit is set and destination to the CPU is picked from the “get oamp_cpuport”  as shown below commands . The destination value in the “get Oamp_cpuport” is given by enabling the Oamp error Traps  like bcmRxTrapOampMyDiscErr .

Here for these traps we use the Multicast , the reason we have only field to specify the destination . So we use multicast. We don’t use unicast because we need to specify Voq ,Encap as we don’t 2 place holders in “Oamp_cpuport”.

OAMP consume the BFD packet

If all the checks pass , the Packet is consumed by the Oamp RX machine  and not punted to CPU .

The Scan processor of the OAMP reads the MEPDB at the minimum rate of entry per 1.6 ms and send the packets to NIF .

Ingress PP Pipeline
The ingress PP pipeline provides two main functions, BFD identification and BFD classification. This article focuses on these functions when Micro-BFD is enabled.
When Micro-BFD is NOT enabled

BFD Single Hop packet identification is performed by the Parser and additional hardwired logic.
BFD Single Hop packet classification is performed by the OAM engine.
o   The OAM engine updates the OAM data provided to the OAM processor.

When Micro-BFD is enabled

BFD Single Hop and Micro-BFD packet identification is performed by the Parser and the Forwarding Lookup (FLP) block.
BFD Single Hop and Micro-BFD packet classification is performed by the FLP block.
o   The Programmable Mapping and Filtering (PMF) block maps the results of the FLP block into the OAM data provided to the OAM processor.
Packet Identification Criteria
BFD Single Hop 
The packet is identified as IP BFD Single-Hop if:
The forwarding header is IP
Parser identification of
o   IP next protocol is UDP

o   UDP destination port = 3784 (configurable)

o   UDP source port is within range 49152 to 65535 (configurable)

TTL = 255
DIP subnet verification should be performed by the internal (OAMP) or external BFD processing point
UDP destination port = 6784 (configurable) instead of 3784

PMF Processing
When a BFD packet is forwarded to the OAM processor, the OAM processor must receive some information on how to process the packet.

§  Trap code indicating the reason for the trap

§  Trap qualifier indicating the OAM-Id of the BFD endpoint

§  OAM-offset indicating the offset (in bytes) of the BFD Packet Data Unit (PDU) from the start of the packet.

When Micro-BFD is NOT enabled, the OAM engine will update this information. However, when Micro-BFD is enabled, the FLP performs the classification and the PMF is used to update this OAM information in the pipeline.

The PMF provides two main functions:

§  Update the information for OAM processing for valid BFD Single Hop/Micro-BFD packets

o   PMF Preseletors

§  Trap code is trap_bfd_o_ipv4 (hw trap id = 0xe3), or

§  Trap code is user_defined_38 (hw trap id = 0xfd)

§  LEM-2nd-Lookup-Found is set

§  PTCH-2 opaque data is 0 (packet was not injected by the OAM processor)

§  Forwarding Type is IPv4-UC

§  Forward Processing Profile is IPv4 BFD Single Hop

o   Two direct extraction DBs are created

§  Update the Trap Action

Unknown macro: {qualifier(16), strength(3), code(8)}
Conditionally qualify on UDP source port range check. If in range:
o   Map the OAM-Id from the LEM-2nd-Lookup-Result into the trap qualifier

o   Set the strength to 7

o   Copy the current trap code

§  Map the OAM-Offset from the FLP Key-B into the OAM action

Set Oam-Up-Mep to 0
Set OAM-Sub-Type to 0
Map the BFD Packet Offset from FLP Key-B into OAM-offset
Set OAM-Stamp-Offset to 0x0
§  Restore original OAM information in case the UDP Source Port is NOT in the expected range.

o   Conditionally qualify on UDP source port range check. If NOT in range:

§  Map Oam-Up-Mep into Oam-Up-Mep

§  Map OAM-Sub-Type into OAM-Sub-Type

§  Map OAM-offset into OAM-offset

§  Map OAM-Stamp-Offset into OAM-Stamp

OAMP Processing">OAMP Processing


BFD packets are processed in 2 cycles. In first cycle, packet is recycled on a well known port (250-core0, 251-core1) before BFD packet processing starts. 

1st cycle:

BFD is treated as an IP packet and will hit the FLP entry which satisfies IP criteria and destination is resolved as recycle port.
The PP port value on which packet is received is extracted in PMF stage and is added as part of stacking header. (PP port value is carried to egress using Stacking header in xr-dev lineup, UDH-2 in r63x lineup, UDH-1 in r62x)
As part of SDK change, FLP PTC profile would be set to 2 which would be an additional preselector criteria for selecting the recycled packet.
2nd cycle:

After recycling the packet, parser would qualify the packet as BFD with parser code to be '0xC'.
BFD FLP program is hit based on the parser context, FLP PTC profile and would set the trap code accordingly.
In PMF stage, BFD packet would be processed and sets the traps & destination accordingly.

