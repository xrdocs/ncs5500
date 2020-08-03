---
published: true
date: '2020-08-03 17:07 +0530'
title: Introduction to NCS5xx and NCS55xx ACL
author: Tejas Lad
excerpt: This is an introduction to ACL feature support on NCS5xx and NCS55xx portfolio
tags:
  - cisco
  - 'NCS5500 '
  - NCS 5500
  - ACL
  - NCS540
  - NCS560
position: hidden
---
{% include toc %}

## Introduction

This documents aims to introduce the ACL support on NCS5xx and NCS55xx product family. It will cover the overview of ACL, ACL Protcols, Types and Support Matrix. We will have series of documents to deep dive into ACL features.

## Overview

![Screenshot 2020-08-03 at 6.03.20 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-03 at 6.03.20 PM.png)

Though everyone would be aware of what an access control list is, I would like to brush up some of the basics before deep-diving into its complex functionalities. 

ACL's can be considered as a ordered list of conditions used to test the network traffic that traverses through the router's interfaces. On the basis of the defined conditions, the router can decide which packets to accept and which to drop. ACL's help in managing the traffic and secure the acsess to and from the network.

ACL's can make permit/deny decisions based on source/destination address, source/destination ports, L3 protocols, L4 port numbers and many other criterias.

ACL's on NCS5xx and NCS55xx portfolio uses the Programmable Mapping and Filtering (PMF) functionality and TCAM (internal/exertnal) in both the Ingress Receive Packet Processing (IRPP) blocks and Egress Receive Packet Processing (ERPP) blocks. The NPU in these platforms are based on the Broadcom family of chipsets.  These chipsets uses a pipeline architecture which has dedicated hardware blocks for performing various functions. For more information on the product family please refer ([NCS5500 Modular Platforms](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-modular-platform-architecture-white-paper.pdf "NCS5500 Modular Platforms") [NCS5500 Fixed Platforms](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-fixed-platform-architecture-white-paper.pdf "NCS5500 Fixed Platforms") [NCS540](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-540-series-routers/datasheet-c78-740296.html "NCS540") [NCS560](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-560-series-routers/datasheet-c78-742029.html "NCS560"))

## Roles of ACL 

| Task                         | Use Cases                                                                                                                                                                                       |
|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Increase Network Performance | ACL's limit the network traffic and help in reducing the network congestion For e.g.heavy traffic like videos can be filtered out if needed which can greatly  increase the network performance |
| Traffic Control              | Certain updates and control traffic can be filtered to preserve the bandwidth                                                                                                                   |
| Secure Access                | Different hosts can be allowed to access only required applications inside the network                                                                                                          |
| Traffic Filter               | ACL's can help to block certain traffic and permit the other. For e.g. email traffic can be allowed whereas Telnet traffic can be blocked                                                       |
| Redirect the traffic         | ACL's can be used to decide the next hop of incoming traffic if needed. e.g ACL based forwarding or ABF                                                                                         |

## High Level WorkFlow

As per the [reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "reference"), below is the high level work flow of ACL 

![Screenshot 2020-08-03 at 6.21.16 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-03 at 6.21.16 PM.png)

## ACL Types 

The following ACL protocol types are supported on the portfolio:

- IPv4
- IPv6
- L2

## ACL Scale

| ACL Type | Direction      | Details                                                    | Scale                                   |
|----------|----------------|------------------------------------------------------------|-----------------------------------------|
| V4       | Ingress        | Max attached unique ACLs per NPU (shared mode)             | 127                                     |
| V4       | Ingress        | Max attached unique ACLs per NPU (unique mode)             | >128                                    |
| V4       | Ingress        | Max ACEs per LC (attached)                                 | 4K                                      |
| V4       | Egress         | Max attached unique ACLs per NPU                           | 255                                     |
| V4       | Egress         | Max ACEs per LC (attached)                                 | 4k Per NPU  only 2k deny ACE is allowed |
| V6       | Ingress        | Max attached unique ACLs per NPU (shared mode/unique mode) | 127                                     |
| V6       | Ingress        | Max attached unique ACLs per NPU (unique mode)             | >128                                    |
| V6       | Ingress        | Max ACEs per LC (attached)                                 | 2K                                      |
| V6       | Egress         | Max attached unique ACLs per NP                            | 255                                     |
| V6       | Egress         | Max ACEs/entries per LC (attached)                         | 2K                                      |
| V4/V6    | Ingress/Egress | Max range ids(used for SRC PORT only) per NPU              | 23                                      |
| L2       | Ingress        | Max unique2 ACLs per NPU (attached)                        | 63                                      |
| L2       | Ingress        | Max ACEs per LC (attached)                                 | 4K                                      | 

```
Note:

1) The ACE scale is a "soft" limit based on the maximum number of ACEs of a given type/direction that we officially support (with headroom).  We require extra space (headroom) available in TCAM to accommodate edit operations.
2) The number of ACEs supported may be further limited by the availability of statistics which is a limited resource 
```

## ACL Interface Support

| ACL Type  | Direction      | Details                                                 | Support |
|-----------|----------------|---------------------------------------------------------|---------|
| IPv4/IPv6 | Ingress/Egress | L3 physical interfaces                                  | Yes     |
| IPv4/IPv6 | Ingress/Egress | Bundle physical interfaces                              | Yes     |
| IPv4/IPv6 | Ingress        | L3 sub-interfaces (main or bundle)                      | Yes     |
| IPv4/IPv6 | Ingress/Egress | BVI                                                     | Yes     |
| IPv4/IPv6 | Egress         | L3 sub-interfaces without HQoS enabled (main or bundle) | No      |
| IPv4/IPv6 | Egress         | L3 sub-interfaces with HQoS enabled (main or bundle     | Yes     |
| IPv4      | Ingress/Egress | Tunnel (IPv4oIPv4, IPv4oIPv4GRE, IPv6oIPv4GRE)          | Yes     |
| IPv6      | Ingress/Egress | Tunnel                                                  | No      |
| L2        | Ingress        | L2 interfaces and sub-interfaces (main or bundle)       | Yes     |
| L2        | Egress         | All interfaces                                          | No      |


## Feature matrix for important ACL Features 


| Feature                                                   | Direction      | Details                                                          |
|-----------------------------------------------------------|----------------|-------------------------------------|
| IPv4 ACL                                                  | Ingress        | IPv4 Ingress ACL                                                 |
| IPv6 ACL                                                  | Ingress        | IPv4 Ingress ACL                                                 |
| IPV4 ACL                                                  | Egress         | IPv4 Egress ACL                                                  | 
| IPv6 ACL                                                  | Egress         | IPv6 Egress ACL                                                  |
| L2 ACL                                                    | Ingress        | L2 ACL                                                           |
| Tunnel v4                                                 | Ingress        | Tunnel (IPv4oIPv4, IPv4oIPv4GRE, IPv6oIPv4GRE) |
| ABF V4/V6                                                 | Ingress        | ACL based forwarding                                             | 
| ABF VRF Aware                                             | Ingress        | IPv4 ABF with VRF + IPv4/v6 address                              |
| ABF Object Tracking V4/V6                                 | Ingress        | IPv4/IPv6 ABF with object-tracking                               | 
| ABF v4/v6 NH over physical, subint, bundle, bundle subint | Ingress        | IPv4/IPv6 NH(s) reachable over physical interface                | 
| ABF v4/v6 NH over tunnel                                  | Ingress        | IPv4/IPv6 NH(s) over tunnels                                     |
| Permit Stats                                              | Ingress        | Permit Statistics                                                |
| UDK V4/V6                                                 | Ingress        | User Defined TCAM Key                                            |
| UDF V4/V6                                                 | Ingress        | User Defined Field                                               | 
| IPv4/v6/L2                                                | Ingress/Egress | ACL MIB                                                          |          
| Hybrid/Compressed ACL V4/V6                               | Ingress        | Support for v4/v6 ingress ACL compression Level 3 only           |
| TTL Support V4/V6                                         | Ingress        | TTL match and TTL set support over ACL                           |
| Fragmentation Permit/Deny ACE Support                     | Ingress        | Fragmentation Permit/Deny ACE Support                            |
| Span V4/V6                                                | Ingress        | SPAN w/ACL-based filtering to choose which traffic gets mirrored |


## ACL Specific hw-module profiles
 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile acl egress layer3 interface-based</mark>
</code>
</pre>
</div>

- Applicable to all LC's
- Used for ACL on BVI interface in egress direction
- Reboot on LC/LC's is needed for the profile to get activated
- After activating this profile, all non BVI interfaces will not support egress ACL configuration



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile stats [acl-permit | qos-enhanced]</mark>
</code>
</pre>
</div>

- Applicable to only J/J+ LC's
- Used to enable ACL permit stats as they are disabled by default
- Reload of LC(s) is needed to activate this hw module profile



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam acl-prefix percent [value]</mark>
</code>
</pre>
</div>

- Applicable to linecards with NL eTCAM type only  
- Not applicable to line cards with OP TCAM type
- Used for External TCAM partitioning for hybrid ACL
- Allocates a certain percentage of the external TCAM for use by compressed ACL
- Not required by uncompressed (compression level 0) ACL
- If the user wants to disable compressed ACL to conserve external TCAM space for other features, the hw-module command can be used with a percentage of 0 for LC NC55-36X100G-A-SE
- For LC's other than NC55-36X100G-A-SE, no space is allocated by default to compressed ACL, so compressed ACL attachment will fail
- Space must be allocated to compressed ACL using the hw-module command and the LC must be rebooted
- This feature is applicable only after 6.3.2



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv4 (qualifiers) [location rack/slot/cpu0]</mark>
</code>
</pre>
</div>

- Applicable only LC's with eTCAM
- Used for User Defined TCAM Keys (UDK) for compressed ACL



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv6 (qualifiers) [location rack/slot/cpu0]</mark>
</code>
</pre>
</div>

- Applicable only LC's with eTCAM
- Used for User Defined Field (UDF) along with UDK



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile acl ingress compress enable [location rack/slot/cpu0]</mark>
</code>
</pre>
</div>

- Applicable only for J2 eTCAM card from 7.0.2
- Enables ACL compression in ingress



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile acl ingress layer3 [ipv4|ipv6] interface-based [location rack/slot/cpu0]</mark>
</code>
</pre>
</div>

- Applicable only for J2 eTCAM card from 7.2.1
- Not using UDK to enable interface based option


## Reference

- [Security ACLs NCS5500](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Security ACLs NCS5500")
- [CCO Documentation](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "CCO Documentation")

## Summary




