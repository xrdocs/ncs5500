---
published: true
date: '2021-03-29 12:45 +0200'
title: IOS XR 7.3.1 Innovations in NCS5500/NCS5700/NCS500 Platforms
author: Nicolas Fevrier
excerpt: What's new in IOS XR731 in term of features and newly supported hardware
position: top
tags:
  - iosxr
---
## Introduction

IOS XR 7.3.1 has been published in late February 2021 and is an ED version for many XR platforms including:  
- NCS5500
- NCS5700
- NCS540
- NCS560

Release notes:
- NCS540: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/73x/b-release-notes-ncs540-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/73x/b-release-notes-ncs540-r731.html)  
- NCS560: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/73x/b-release-notes-ncs560-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/73x/b-release-notes-ncs560-r731.html)
- NCS5500: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html)

Software download for NCS5500:  
[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html)

![download731.png]({{site.baseurl}}/images/download731.png){: .align-center}

We asked a few colleagues to help documenting the new improvements brought by the 7.3.1 version. It will be split in software innovations on one side (Segment Routing, SRv6, EVPN, Multicast, QoS, Security, ...) and new supported hardware (chassis, power supply, new line cards and new fixed-form-factor products).

## Segment Routing IPv6 (SRv6)

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/Dna59DZlLmQ?autoplay=1" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jakub Horn presents and demonstrates in his lab:
- SRv6 uSID
- uSID config
- uSID ISIS config
- uSID BGP L3VPN config
- TI-LFA and uLoop Avoidance config
- Performance Measurement config
- FlexAlgo config

The configuration used for the demos are available here:  
[https://www.segment-routing.net/tutorials/srv6-731-features/](https://www.segment-routing.net/tutorials/srv6-731-features/)

## EVPN

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/3eoDB7uia6k" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jiri Chaloupka and Lampros Gkavogiannis detail the following features:  
- EVPN Head-End
- EVPN Convergence
- Fast ReRoute
- MAC Mobility
- Mass Withdraw
- Edge Failure Fast ReRoute
- EVPN Load Balancing
- SFA Single Flow Active
- Next-Hop Tracking for DF Election
- NTP sync
- Multicast MultiHoming

## Quality of Service

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/lvhHNdXEC_c" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Paban Sarma provides details on new QoS features:  
- Shared-Policy Instances
- Policy-map templates and uniqueness

## Security

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/O6tTu2Nidj8" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Rakesh Kandula presents the latest improvements in IOS XR security brought in IOS XR 7.3.1:  
- SSD encryption (note: it's limited to the platform running XR7)
- X.509v3 Certificate based SSH authentication

## MPLS Segment Routing

This section is split itself in three parts

### IGPs

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/5myoWuSioGY" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jose Liste covers a lot of topics in this 30min long video:  

- OSPF LFA/TI-LFA for FlexAlgo
- Rounding of Min-Delay Values
- OSPF Conditional Advertisement
- ISIS TI-LFA Protection of Unlabeled IPv6 Prefixes
- Inter-Level SRMS Advertisement Propagation

### Performance Monitoring and Traffic-Engineering

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/KkdX7CL7bGU" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

- Named Profile
- SR Policy Delay Measurement with Loopback-mode
- SR Policy Liveless Monitoring
- SR-TE Cumulative-Metric Bound

### Dynamic SR P2MP Policies

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/oG11YKMzxqU" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

- Dynamic SR P2MP policies theory and lab demo

## Optics

Coming soon

## New products

### Chassis commons

Coming soon

### NC55-32T16Q4H-A Line Card

Coming soon

### NC57-36H-SE

Coming soon

### NCS57B1-6D24 / NCS57B1-5DSE

Coming soon
