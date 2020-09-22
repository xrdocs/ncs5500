---
published: true
date: '2020-09-22 13:46 +0200'
title: NCS-5500 IOS XR 7.2.1 New Features
author: Nicolas Fevrier
excerpt: All the IOS XR 7.2.1 Innovations
position: hidden
---
{% include toc icon="table" title="XR 7.2.1 New Features" %} 

You can find more content related to NCS-5500 and NCS-5700 following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Following the release of the IOS XR 7.2.1, we decided to gather information on the diverse new features added with this new version.  
It should be considered a complement of the existing Release Notes giving the opportunity to multiple specialists to tell us a bit more about these innovations, sometimes illustrated by a short demo.

To download this IOS XR 7.2.1 release: [https://software.cisco.com/download/home/286291132/type/280805694/release/7.2.1](https://software.cisco.com/download/home/286291132/type/280805694/release/7.2.1)  
Release Notes: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/72x/release/notes/b-release-notes-ncs55k-r721.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/72x/release/notes/b-release-notes-ncs55k-r721.html)

Many thands to the presenters:  
- Jose Liste
- Phil Bedard 
- Jiri Chaloupka
- Vaishnavi Kolimi
- Bernhard Stascheit
- Vincent Ng
- Paban Sarma
- Neeraj Garg 
- Tejas Lad
- Rakesh Kandula
- Akshaya Kumar Sankaran 
- Nicolas Fevrier

## Videos

Considering the amount of ground to cover, we split the presentation in multiple parts.

### Part 1

[https://www.youtube.com/watch?v=eURgr9hgA2I](https://www.youtube.com/watch?v=eURgr9hgA2I)

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/eURgr9hgA2I?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}
  
  
Covers many features around Segment Routing and EVPN.

### Part 2

[https://www.youtube.com/watch?v=PedQo7DRDf4](https://www.youtube.com/watch?v=PedQo7DRDf4)

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/PedQo7DRDf4?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}
  
  
Covers both platform (NCS55A1-24Q6H-SS and new optics) and new features (WAN-Phy, QoS, ECMP, ACLs, ...)

### Part 3

[https://www.youtube.com/watch?v=9IYER-jIZ4Y](https://www.youtube.com/watch?v=9IYER-jIZ4Y)

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/9IYER-jIZ4Y?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}
  
  
Covers other features including Security, DHCP, BVI, BFD, file management and bridge domains scale.

### Part 4

Will be covered in a dedicated post.

## Features in details

### SR IGP: Flex Algo Prefix Metric

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=110s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=110s)

### SR IGP: Rounding of Min-Delay Values

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=254s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=254s)

### SR IGP: OSPF TI-LFA Weighted Remote SRLG

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=345s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=345s)

### SR IGP - TI-LFA GRE as Outgoing Interface for Backup Path

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=447s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=447s)

### SR-PCE - Flex Algo-aware Path Computation 

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=561s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=561s)

### SR-PCE - SRTE to BGP-LU Interworking

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=726s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=726s)

### SR-PCE -MPLS-TE LSP Support

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=838s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=838s)

### SRTE - Per-Flow ODN / AS

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=920s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=920s)

### SR EPE - Manual BGP Peering SIDs for Segment Routing Egress Peer Engineering and Segment Routing BGP Peer-Set SID

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=1200s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=1200s)

### SR ODN - On-Demand Next Hop for EVPN and EVPN VPWS

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=1405s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=1405s)

### EVPN E-Tree Using RT Constraints

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=1537s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=1537s)

### EVPN Automatic Unfreezing of MAC and IP Addresses

Direct link: [https://www.youtube.com/watch?v=eURgr9hgA2I&t=1675s](https://www.youtube.com/watch?v=eURgr9hgA2I&t=1675s)

### New product: NCS55A1-24Q6H-SS

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=54s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=54s)

### New supported optics

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=156s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=156s)

### WAN-Phy on NC55-MPA-12T-S

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=239s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=239s)

### QoS: Share Policer

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=300s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=300s)

### QoS: Conform Aware Hierarchical Policy

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=452s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=452s)

### QoS: Egress Shaper Burst

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=579s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=579s)

### ECMP: GTP Tunnels over MPLS

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=652s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=652s)

### ECMP overs MPLS-TE

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=752s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=752s)

### Chained/Common ACLs

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=846s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=846s)

### ACL: Support of IPv6 Fragments (for Jericho2 platforms)

Direct link: [https://www.youtube.com/watch?v=PedQo7DRDf4&t=1046s](https://www.youtube.com/watch?v=PedQo7DRDf4&t=1046s)

### Security: Password policy for user secret

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=61s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=61s)

### Security: 802.1x Port Control

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=122s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=122s)

### Security: Guideline for enabling FIPS

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=191s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=191s)

### DHCP:: v4 and v6 options on BVI

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=266s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=266s)

### DHCP: v6 client on management interface

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=310s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=310s)

### DHCP: v6 options 15, 16, 23 and 24

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=345s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=345s)

### DHCP Lab Demo

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=371s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=371s)

### BVI with 2 VLAN tags

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=474s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=474s)

### BFD for VRRP and HSRP

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=582s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=582s)

### BFP over unnumbered v4 Bundle 

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=788s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=788s)

### File Mirroring

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=684s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=684s)

### Bridge Domain scale improvement

Direct link: [https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=743s](https://www.youtube.com/watch?v=9IYER-jIZ4Y&t=743s)






