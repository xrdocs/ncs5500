---
published: false
date: '2020-12-21 13:04 +0530'
title: 'Y.1564 SADT on NCS : Part1'
author: Paban Sarma
excerpt: >-
  First Part of the Tutorial series explaining Y.1564 SADT on NCS 5500 and NCS
  500 routers 
tags:
  - iosxr
---

|Revision History|

## Overview  
The ITU-T recommendation Y.1564 defines an out-of-service test methodology to confirm the proper configuration and performance of an Ethernet service prior to customer delivery and covers the case of both a point-to-point and point-to-multipoint topology. This service activation acceptance testing of Ethernet-based services can be implemented as a test function inside of a network element. This particular functionality was introduced on Cisco NCS 500 and NCS 5500 series routers starting release IOS XR 7.1.x. 

The objective of this series of articles is to capture the Y.1564 capabilities of NCS 5500 and NCS 500 series routers. we will dicsuss the implementation and various use cases and their configurations & verifications. This part includes concpets, supported scenarios & configuration examples to demonstrate how operators can run Y.1564 SADT on the box.

## Y.1564 Concepts:
### Traffic Generation and Measurement 
There are various operations as per Y.1564 viz.
- Traffic Generation Mode:   
 _The device under test (DUT) generates the traffic and sends it out on the interface where service   has been provisioned. This eliminates the need for an external probe_
- Passive Measurement Mode:   
_Here the DUT measures the traffic received on the service interface in order to verify the proper service configuration. This mode is not available in NCS 500 and NCS 5500 products._
- Two way statistics collection Mode:   
_In this mode, traffic generation and  all measurements are done locally on the DUT. Traffic is looped back on the far end after MAC swap. Based on the return traffic this mode calculates various statistic like throughput, loss etc. For this mode, the remote end needs to be properly configured to loopback the traffic sent towards it. The Y.1564 implementation on NCS 500 and NCS 5500 routers implements the two-way mode._   

The direction of Traffic Generation can be internal or external. In internal mode the traffic is generated at the UNI and forwarded towards the network (via the service). In external mode traffic is sent out of the interface.

### Target Services
The services running on the network can be both Layer2 and Layer3. The Y.1564 feature on NCS 500 and NCS 5500 routers addresses only the layer 2 point to point services, viz.
- Layer 2 local cross-connects
- Layer 2 VPWS (T-LDP PW)
- EVPN-VPWS

These L2 services can be configured on UNI based on main interface or sub-interface (both physical and bundle). 
### Traffic Profiles
With the current implementation of Y.1564 on NCS 500 and NCS 5500 routers can only generate layer2 traffic. The following different parameter can be specified in the Y.1564 test profile.

- Outer COS 
- Inner COS
- DEI (for color aware flows)
- Packet Size
  - Fixed size (range)
  - EMIX pattern (defined in Y.1564)

The following table shows the packet size for Y.1564 SADT on the NCS platforms.

|a | b | c | d | e | f | g | h | u |
|64 | 128 | 256 | 512 | 1024 | 1280 | 1518 | MTU | user defined | 


