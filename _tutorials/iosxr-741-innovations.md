---
published: true
date: '2021-09-09 23:04 +0200'
title: IOS XR 7.4.1 Innovations in NCS5500/NCS5700/NCS500 Platforms
author: Nicolas Fevrier
excerpt: New features and hardware supported in IOS XR 7.4.1
position: top
---
{% include toc icon="table" title="XR 7.4.1 New Features and Hardware for DNX Platforms" %} 


<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/qvczWr2Vu_U" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

## Introduction

IOS XR 7.4.1 has been published in August 2021 and is an ED version for many XR platforms including:  
- NCS5500
- NCS5700
- NCS540
- NCS560

Release notes:
- NCS540: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/74x/b-release-notes-ncs540-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/74x/b-release-notes-ncs540-r741.html)  
- NCS560: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/74x/b-release-notes-ncs560-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/74x/b-release-notes-ncs560-r741.html)
- NCS5500: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/74x/release/notes/b-release-notes-ncs5500-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/74x/release/notes/b-release-notes-ncs5500-r741.html)

Software download for NCS5500:  
[https://software.cisco.com/download/home/286291132/type](https://software.cisco.com/download/home/286291132/type)

![download.png]({{site.baseurl}}/images/download.png){: .align-center}

Note: two flavors of IOSXR are now available for NCS5500/NCS5700 platforms

## Software Images

### XR 64-bit

In this section of the download center, you'll find software images for all platforms, fixed or modular running IOS XR 64-bit, with the particular exception of the NCS-57B1 platforms.

![5500-soft.png]({{site.baseurl}}/images/5500-soft.png){: .align-center}

### XR7

In current release, one single platform type in the NCS5500/NCS5700 series is running the new version of IOS XR named "XR7" ("LNT" in the show commands). That's the NCS57B1-6D24 and NCS57B1-5DSE. And image can be found in this section of the download center (under 5700):

![5700-soft.png]({{site.baseurl}}/images/5700-soft.png){: .align-center}

Note: Even the latest platform NCS57C3-MOD is running XR 64-bit and not XR7.
{: .notice--info}

Please refer to the release notes and installation guide to get details on the upgrade process.

## New Features in IOS XR 7.4.1

We asked a few colleagues to help documenting the new improvements brought by the 7.3.1 version. It will be split in software innovations on one side (Segment Routing, SRv6, EVPN, Multicast, QoS, Security, ...) and new supported hardware (chassis, power supply, new line cards and new fixed-form-factor products).

### ACL

Tejas Lad demonstrates the improvements brought by J2 ASIC for egress ACL and statistics.  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/NJHSheNfDZw" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

With Jericho2 platforms, a series of limitations present in Jericho/Jericho+ have been lifted, for example:  
- We don't need to use a specific hardware module profile configuration to allocate counters for ACL permit packets. "hw-module profile stats acl-permit" is not used anymore.  
- In Jericho and Qumran-MX platforms with NL12k eTCAM, it could be necessary to recarve the resource with "hw-module profile  tcam acl-prefix percent <0-100>". It's all dynamic on Jericho+ with OP and Jericho2 with OP2 eTCAMs, no config is needed
- We don't recycle packets for ACLv6 in egress direction like it was the case in Jericho and Jericho+ platforms, the packet treatment with J2 is done in one pass.
- We support egress ACLv6 on BVI interfaces, something not supported on J/J+ platforms.

Tejas also published another article here with plenty of details:  
[https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/)

In IOS XR 7.4.1, we complete this list of improvements with more fields.

| Match Parameter | IPv4 support pre-741 | IPv4 support in 741 | IPv6 support pre-741 | IPv6 support in 741 |
|:---:|:---:|:---:|:---:|:---:|
| Source Address | Yes | Yes | Yes | Yes |
| Destination Address | Yes | Yes | Yes | Yes |
| Source Port | Yes | Yes | Yes | Yes |
| Destination Port | Yes | Yes | Yes | Yes |
| Protocol/ Next Header | Yes | Yes | Yes | Yes |
| Precedence/DSCP | Yes | Yes | No | Yes |
| Packet Length | No | Yes | No | Yes |
| TCP control Flags | Yes | Yes | No | Yes |
| Fragment bit | Yes | Yes | No | Yes |
| Extended Header | N/A | N/A | No | Yes |
| Destination System port | Yes | Yes | Yes | Yes |

### Security

Rakesh Kandula presented new security feature: Chip Guard on NCS540.  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/2fwIZRRJPSw" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

Let's first describe two essential components into every IOS XR router:  
- TAm: the Trust Anchor module chip integrated in all recent device
- Cisco Secure Boot: the chain ensuring the integrity of the entire boot process

![Secure-boot-process.png]({{site.baseurl}}/images/Secure-boot-process.png){: .align-center}

Also these components are not enough if the hardware itself has been replaced by malware infected parts (CPU or NPU). And that's the purpose of this new feature: Chip Guard introduced now in our access platforms: NCS540.

Chip Guard feature is triggered during the BIOS part of the boot sequence.  
During manufacturing:
- the SHA-256 hash of the Electronic Chip ID (ECID) of the CPU and NPU are calculated
- These hashes are then programmed inside the TAm chip
- The programmed hash values form the ImprintDB inside the TAm chip
- The ImprintDB cannot be modified during runtime

![Manufacturing-diag-process.png]({{site.baseurl}}/images/Manufacturing-diag-process.png){: .align-center}

![ImprintDB.png]({{site.baseurl}}/images/ImprintDB.png){: .align-center}

When the router boots:  
- BIOS reads the ECID of the chips and computes their hashes
- Each of the hashes is then extended into a PCR inside TAm chip
- These set of observed hashes forms the ObserveDB

![Step1.png]({{site.baseurl}}/images/Step1.png){: .align-center}

- BIOS fetches the factory programmed hash values from imprintDB
- The hash values are compared with the ObserveDB generated in the previous step
- BIOS continues with boot process if and only if the hashes match

![Step2.png]({{site.baseurl}}/images/Step2.png){: .align-center}

### QoS

Paban Sarma introduces PPS-based policer.  
Coming soon

### BFD

Tejas Lad presents the BoB-BLB co-existence benefits.  
Coming soon

### Scale improvements

Nicolas Fevrier presents the scale improvements directly available in J2 native mode for BGP Flowspec, ARP scale and ECMP FEC.  
Coming soon


### Segment Routing

Jose Liste describes in details the new features introduced in SR MPLS world.  
Coming soon


### Timing

Vincent Ng presents timing features introduced in XR 7.4.1.  
Coming soon


### Misc

Coming soon

## New hardware in IOS XR 7.4.1

Placeholder for the new platforms, MPA and IM cards.
