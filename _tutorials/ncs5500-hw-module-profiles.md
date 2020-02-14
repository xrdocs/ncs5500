---
published: true
date: '2020-02-14 16:39 +0100'
title: NCS5500 Hw-module Profiles
author: Nicolas Fevrier
excerpt: WIP document describing the various options behind the NCS5500 hw-module CLI
tags:
  - iosxr
  - dnx
position: hidden
---
{% include toc icon="table" title="NCS5500 Hw Profiles" %} 

|   |   |
|:-----:|:-----:|
| 2020-Feb-14 | Document Creation |

You can find more content related to NCS5500 including routing memory management, URPF, ACLs, Netflow following this [link](https://xrdocs.io/ncs5500/tutorials/).

This article is meant to be updated regularly, consider it a constant "work in progress".

## Introduction

During the 4 years of its existence, the NCS5500 has been used in a constantly growing number of network roles. The NCS 5500 resources (databases where we store prefixes, nexthop, counters, ...) have been optimized to accommodate the specific requirements of these networking roles.  
To enable these particular resource optimizations, we carved the memories in specific ways, via "hardware profiles".

This document aims at listing all these options and clarify:
- what they do and where they can be used
- which platforms can use them
- when they have been introduced
- what are the side effects of enabling them, if any

If not specifically mentioned, consider that activation of a new hw-module config will require a system or line card reload.

**Acknowledgements**

Many thanks to:
- Neelu Jethani
- Jisu Bhattacharya
- Vincent Ng
- Anup Kumar Vasudevan
- Rajeev Mishra
- Aleksandar Vidakovic
- Neeraj Garg

## Hardware Module CLI hierarchy

For this article, we use IOS XR 6.6.3 and 7.0.2. The document will be updated with new entries with new releases.  

### Graphical view of the 6.6.3 structure

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663#sh ver

Cisco IOS XR Software, Version 6.6.3
Copyright (c) 2013-2019 by Cisco Systems, Inc.

Build Information:
 Built By     : hlo
 Built On     : Fri Dec 13 17:40:12 PST 2019
 Built Host   : iox-lnx-029
 Workspace    : /auto/srcarchive15/prod/6.6.3/ncs5500/ws
 Version      : 6.6.3
 Location     : /opt/cisco/XR/packages/

cisco NCS-5500 () processor
System uptime is 2 weeks 6 days 14 hours 56 minutes

RP/0/RP0/CPU0:NCS5500-663#conf
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module ?
  fib               Forwarding table to configure
  oversubscription  Configure oversubscription
  profile           Configure profile.
  quad              Configure quad.
  route-stats       Configure multicast per-route statistics
  service           Configure service role.
  subslot           Configure subslot h/w module
  tcam              Configure profile for TCAM LC cards
  vrrpscale         to scale VRRP sessions
RP/0/RP0/CPU0:NCS5500-663(config)</code>
</pre>
</div>

![hw-module-663.png]({{site.baseurl}}/images/hw-module-663.png){: .align-center}

### Graphical view of the 7.0.2 structure

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702#sh ver
Cisco IOS XR Software, Version 7.0.2.18I
Copyright (c) 2013-2019 by Cisco Systems, Inc.

Build Information:
 Built By     : ahoang
 Built On     : Tue Nov 19 16:44:39 PST 2019
 Built Host   : iox-ucs-027
 Workspace    : /auto/iox-ucs-027-san2/prod/7.0.2.18I.SIT_IMAGE/ncs5500/ws
 Version      : 7.0.2.18I
 Location     : /opt/cisco/XR/packages/
 Label        : 7.0.2.18I

cisco NCS-5500 () processor
System uptime is 4 weeks 6 days 12 hours 8 minutes

RP/0/RP0/CPU0:NCS5500-702#conf
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module ?
  ains              Configure AINS Params
  fib               Forwarding table to configure
  oversubscription  Configure oversubscription
  port-range        Configure port range
  profile           Configure profile.
  quad              Configure quad.
  route-stats       Configure multicast per-route statistics
  service           Configure service role.
  shut              shutdown the hw-module
  stats-fpga        Configure h/w module
  subslot           Configure subslot h/w module
  tcam              Configure profile for TCAM LC cards
  unshut            Unshut the hw-module
  vrrpscale         to scale VRRP sessions
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![hw-module-702.png]({{site.baseurl}}/images/hw-module-702.png){: .align-center}

Now, let's review these profiles individually. We will define their role, the type of platforms using them, and potentially the conflicts with other profiles.

##  fib

### dlb

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module fib dlb level-1 enable ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

"Destination-based Load balancing" has been introduced in 6.6.2 and is completed in 7.1.1.  
It's another solution to address the 4K ECMP FEC limitation with software based pre-selection of the path. In this approach, one path out of available multipaths is selected based on software hash on destination IP before prefix is programmed in hardware. Data plane is programmed with selected single path. So traffic is virtually distributed on available paths by pre-selection of path based on per-prefix hash.  
This profile enables DLB for IGP with LDP, IGP with SR, L2VPN PW over IGP, BGP LU over IGP, BGP L3VPN over IGP, TI-LFA/LFA/RLFA at IGP but not for IGP over SRTE nor BGP PIC Edge.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/71x/b-routing-cg-ncs5500-71x/b-routing-cg-ncs5500-71x_chapter_01000.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/71x/b-routing-cg-ncs5500-71x/b-routing-cg-ncs5500-71x_chapter_01000.html)

### ipv4 / ipv6

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ?
  dlb   Destination Based Load balancing
  ipv4  Configure ipv4 protocol
  ipv6  Configure ipv6 protocol
  mpls  Configure mpls protocol
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv4 ?
  scale  Configure scale mode for no-TCAM card
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv4 scale ?
  host-optimized-disable  Configure Host optimization by default
  internet-optimized      Configure Intetrnet optimized
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv4 scale internet-optimized ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv4 scale host ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv6 ?
  scale  Configure scale mode for no-TCAM card
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv6 scale ?
  internet-optimized-disable  Configure by default Intetrnet optimized
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib ipv6 scale internet ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![01.png]({{site.baseurl}}/images/01.png){: .align-center}

These profiles will dictate how we distribute IPv4 and IPv6 prefixes in the different databases (LPM or LEM) depending on their prefix length. Not effective when using external TCAM.  
They are mandatory if we need to store large routing tables (ie full internet view) and/or if want to configure URPF.  

Default mode is "host-optimized" for IPv4 and "internet-optimized" for IPv6. This hardware profile is only relevant for systems/LC with the no eTCAM ("base" systems) using Jericho and "Jericho+ with Jericho-scale" (with the 256k-350k large LPM). It's not recommended for NCS55A1-24H or NCS55A1-48Q-6H, based on a Jericho+ with large LPM (1M-1.5M v4 entries).

_External documentation_:  
- [https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/](https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/)
- [https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/](https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/)
- [https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/](https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/)
- [https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/](https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/)
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/62x/b-system-security-cg-ncs5500-62x/b-system-security-cg-ncs5500-62x_chapter_01001.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/62x/b-system-security-cg-ncs5500-62x/b-system-security-cg-ncs5500-62x_chapter_01001.html)

Please pay attention to the form of the command since it could lead to confusions: for IPv4 it's internet-optimized while it's internet-optimized**-disable** for IPv6.
{: .notice--info}

### mpls

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib mpls ?
  label  Configure MPLS label convergence optimization for LDP/SR labels
  ldp    Configure signalling protocol for MPLS
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

**hw-module fib mpls ldp lsr-optimized**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib mpls ldp lsr-optimized ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

When using an MPLS network, you bind specific prefixes to labels for the IP-to-MPLS case, and also you bind  ingress labels to egress labels for the LSR role (MPLS-to-MPLS case).  
In the NCS5500, several resources are used to store this information. Among them, the FEC database is solicited. LEM and EEDB are also important in this discussion but we will try to simplify it to focus only on the FEC part.  

When the destination of the packet is resolved via a single path, the information is stored in this 124k-entry large database. But when the destination is resolved via multiple equal cost paths (ECMP case), the information is stored in a sub-block of the FEC table named ECMP FEC.  
This zone can accommodate 4k entries.

By default, the system will allocate 3 entries for each prefix bound to a label:
- one for the IP to MPLS case
- one for the MPLS to MPLS case
- one for the case where the next hop is made of several paths, some with LDP, some others with IP-only (referred as EOS0/1 case, it's usually a transcient situation).
That reduces the overall number of prefixes associated to labels to 1,300 (you'll find many places mentioning the supported number is 1,000).

When we position NCS5500 as a pure LSR role, the first and third allocation are not necessary. That's the purpose of this hardware profile: extending the support to 3000+ prefixes bound to labels (with up to 16 way ECMP paths). This CLI enables creation of "Push/Swap" Shared MPLS encap i.e. same encap can be used for Label Push or Label Swap.

EVPN services are not supported with this profile.

Note: in an IGP domain, only the router's loopbacks need to be bound to labels. It's a best practise to use filters to reduce the IGP to MPLS relationship.
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/mpls/63x/b-mpls-cg-ncs5500-63x/b-mpls-cg-ncs5500-63x_chapter_0101.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/mpls/63x/b-mpls-cg-ncs5500-63x/b-mpls-cg-ncs5500-63x_chapter_0101.html)

**hw-module fib mpls label lsr-optimized**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib mpls ldp ?
  lsr-optimized  Configure optimization for LSR role
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module fib mpls label lsr-optimized ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![08.png]({{site.baseurl}}/images/08.png){: .align-center}

ECMP optimization for /32 prefixes - when all paths have same label, the common label stored in the LEM so we don't have to consume EMCP FEC entries.
In 6.5.x: no services could work with this profile (no L2VPN/L3VPN as the "transport" label won't be pushed on the traffic). Starting from 7.1.1, L3VPN supported but no L2.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/segment-routing/71x/b-segment-routing-cg-71x-ncs560/b-segment-routing-cg-71x-ncs560_chapter_011.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/segment-routing/71x/b-segment-routing-cg-71x-ncs560/b-segment-routing-cg-71x-ncs560_chapter_011.html)

Note: lsr-optimized mode was introduced in 6.5.x and was not supported on J+ w/ eTCAM systems initially, this limitation is removed starting from 7.1.1.
{: .notice--info}

Note2: this profile can not be used with the internet-optimized.
{: .notice--info}

## oversubscription

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize ?
  cos       CoS values between 0-5
  untagged  Prioritize untagged packets
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize cos ?
  0-5  configure CoS number 0-5
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize cos 0 ?
  interface  Interface name
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize cos 0 interface hu0/0/0/0 ?
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize untagged ?
  interface  Interface name
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize untagged interface hu0/0/0/0 ?
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module oversubscription prioritize untagged interface hu0/0/0/0</code>
</pre>
</div>

Features developped for NCS540, not NCS5500.  

## port-range

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range ?
  0-35  configure start port
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range 0 ?
  0-35  configure end port
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range 0 1 ?
  location  fully qualified location specification
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range 0 1 location 0/4/CPU0 ?
  mode  port mode 40-100, 400, 2x100, 4x10-4x25
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range 0 1 location 0/4/CPU0 mode ?
  WORD  port mode 40-100, 400, 2x100, 4x10-4x25
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

Feature introduced to assign port roles in NC57-18DD-SE line cards (Vigor-SE).  

On NC57-18DD-SE line cards, the ports from 0 to 17 and 24-29 should be configured in pairs. They below to a same Reverse Gear Box / CDR5 cage that can handle up to 400Gbps. If the port n is 400G, the port n+1 is disabled. Default mode is 40-100.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5500-702(config)#do sh int brief | i "(0/3/0/28|0/3/0/29)"
         Hu0/3/0/28  admin-down  admin-down               ARPA  1514  100000000
         Hu0/3/0/29  admin-down  admin-down               ARPA  1514  100000000
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module port-range 28 29 loc 0/3/CPU0 mode 400
RP/0/RP0/CPU0:NCS5500-702(config)#commit
LC/0/3/CPU0:Feb  2 22:37:54 : ifmgr[163]: %PKT_INFRA-LINK-3-UPDOWN : Interface HundredGigE0/3/0/29, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:54 : ifmgr[163]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface HundredGigE0/3/0/29, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:54 : ifmgr[163]: %PKT_INFRA-LINK-3-UPDOWN : Interface HundredGigE0/3/0/28, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:54 : ifmgr[163]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface HundredGigE0/3/0/28, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:56 : ifmgr[163]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/3/0/28, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:56 : ifmgr[163]: %PKT_INFRA-LINK-3-UPDOWN : Interface FourHundredGigE0/3/0/28, changed state to Down
LC/0/3/CPU0:Feb  2 22:37:56 : ifmgr[163]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface FourHundredGigE0/3/0/28, changed state to Down
RP/0/RP0/CPU0:Feb  2 22:37:57 : config[67778]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'root'. Use 'show configuration commit changes 1000000037' to view the changes.
RP/0/RP0/CPU0:NCS5500-702(config)#do sh int brief | i "(0/3/0/28|0/3/0/29)"
         FH0/3/0/28        down        down               ARPA  1514  400000000
RP/0/RP0/CPU0:NCS5500-702(config)#
</code>
</pre>
</div>

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/interfaces/71x/b-interfaces-hardware-component-cg-71x-ncs560/preconfiguring_physical_interfaces.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/interfaces/71x/b-interfaces-hardware-component-cg-71x-ncs560/preconfiguring_physical_interfaces.html)

## profile

### acl egress

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile acl ?
  egress  egress acl
  ipv6    ipv6 protocol specific options
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile acl egress ?
  layer3  egress layer3 acl
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile acl egress layer3 ?
  interface-based  egress layer3 interface-based acl
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile acl egress layer3 interface-based ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

Introduced in IOS XR 6.1.4, this profile is specifically designed to enable L3 egress ACL over BVI.  

_Restrictions_: Once this profile is enabled, it's not possible to configure egress ACL over non-BVI interfaces (Physical included).  

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/61x/b-ncs5500-ip-addresses-configuration-guide-61x/b-ncs5500-ip-addresses-configuration-guide-61x_chapter_01.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/61x/b-ncs5500-ip-addresses-configuration-guide-61x/b-ncs5500-ip-addresses-configuration-guide-61x_chapter_01.html)

### acl ingress

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ?
  egress   egress acl
  ingress  ingress acl
  ipv6     ipv6 protocol specific options
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ingress ?
  compress  Specify ACL compression in hardware
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ingress compress ?
  enable  Enable ACL compression in hardware
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ingress compress enable ?
  location  Location of acl config
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

Limited to the NC57-18DD-SE line cards (based on Jericho2 with eTCAM OP2), this profile will enable the hybrid ACL configuration.  

### acl ipv6

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ?
  egress   egress acl
  ingress  ingress acl
  ipv6     ipv6 protocol specific options
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ipv6 ?
  ext-header  ipv6 extension header related options
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ipv6 ext-header ?
  permit  allow permit of extension header packets
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile acl ipv6 ext-header permit ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

Feature available on 6.6.3 and 7.0.1.  

In previous implementation, ipv6 header parser can't identify next protocol properly when one or more extension headers are present. This means, if ipv6 extension header is present, it will not be able to apply security ACLs properly based on ip protocols and other L4 information like L4 ports, TCP flags etc.  
When these extension headers are detected, these packets are sent to control plane CPU for further processing and applying the security ACLs. This mean, these packets will not be processed at full rate, which can be anything from 150Mpps to 835Mpps based on ASIC type, but only at 100 packets per sec. Any more packets of this type will be dropped.  
This behavior of sending packet to CPU is enabled by default, but in case user wants to disable this special handling of extension headers, they can enable the hardware profile.  
With this, they don't have to insert permit rules in each ACLs. All the packets with extension headers will bypass security ACLs and will be permitted. This CLI can be configured or de-configured anytime during router operation and does not need any restart of system.  

Note: Can be enabled and disabled without requiring a system reload.
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html)

### bundle-scale

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile bundle-scale ?
  1024  Max 1024 trunks, Max 16 members
  256   Max 256 trunks, Max 64 members
  512   Max 512 trunks, Max 32 members
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

In IOS XR 6.5.1, we introduced this hardware profile to offer more flexibility in the distribution "number of bundles vs numbers of bundle members". Some customers wanted more bundle interfaces with few ports or vice versa fewer bundle interfaces with more port members.

| # Bundles | Members / Bundle |
|:------:|:------:|
| 256 | 64 |
| 512 | 32 |
| 1024 | 16 |

Note: "hw-module profile qos max-trunks <256/512/1024>" is replaced by "hw-module profile bundle-scale <256/512/1024>" from 6.5.1 release onwards
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html#wp3930832296](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html#wp3930832296)

### flowspec

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile flowspec ?
  v6-enable  Configure support for v6 flowspec
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile flowspec v6-enable ?
  location  Location of flowspec config
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![04.png]({{site.baseurl}}/images/04.png){: .align-center}

NCS 5500 supports BGP FlowSpec starting from IOS XR 6.5.1. Only the J+ platforms with eTCAM (-A-SE) are supporting this feature.

Details on the BGP FlowSpec implementation are available in this support forum post:  
[https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)

Once you enable this profile, due to the potentially very long key necessary to perform the matching part of the BGP FSv6 rule, it has been decided to reduce the PPS for the all data paths to 700MPPS (and not only IPv6 packets).

_External documentation_:  
- [https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)
- [https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/](https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/)

### load-balance algo

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile load-balance algorithm ?
  L3-only                        L3 Header only Hash.
  gtp                            GTP optimized.
  gtp-mpls                       GTP over MPLS optimized hash.
  ip-tunnel                      IP tunnel optimized.
  layer2                         Layer 2 optimized.
  mpls-safe-speculative-parsing  MPLS safe Speculative parsing.
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

There are 5 modes which can be configured. The default (i.e. with no configuration) selects the algorithm which has been running in previous releases.
- ip-tunnel allows the hashing algorithm to use the outer IPv4 GRE header even when doing an IP tunnel decapsulation. 
- layer2 allows the hashing algorithm to be able to use the inner IP header information when doing layer 2 forwarding and the inner payload is MPLS.
- gtp allows hashing based upon the tunnel id in GTP-U (UDP DST 2152) packets. (6.5.1 release or later). When this option is selected, the hashing is based on ip[v4/v6] (src, dest, protocol) + udp (src, dest) + tunnel endpoint id).
- gtp-mpls allows hashing based upon the tunnel id in GTP-U (UDP DST 2152) packets. (7.2.1 release or later). When this option is selected, the hashing is based on tunnel endpoint id.
- mpls-safe-speculative-parsing mode was introduced (6.5.3 release or later) for cases where the network has MPLS carrying L2VPN packets without control words. In this case, if the first nibble of the MAC DA address is a 4 or 6, the rest of the packet could be interpreted incorrectly as IPv4 or IPv6 header. In this mode, the algorithm uses a subset of the IPv4 and IPv6 header for load balancing. It does this for MPLS 1-6 labels for IPv6 and MPLS 4-6 labels for IPv4. For MPLS 1-3 labels and IPv4, the hardware can correctly determine whether it’s an IPv4 header or not (uses the checksum).
 
These profiles can be activated without requiring a reload of the system or the line card.
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/b-ncs5500-interfaces-cli-reference/b-ncs5500-interfaces-cli-reference_chapter_011.html#wp2176473466](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/b-ncs5500-interfaces-cli-reference/b-ncs5500-interfaces-cli-reference_chapter_011.html#wp2176473466)

### netflow ipfix315-enable

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile netflow ?
  fpc-enable       Netflow full packet capture enable
  ipfix315-enable  IPFIX 315 enable
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile netflow ipfix315-enable ?
  location  Location of NETFLOW config
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![05.png]({{site.baseurl}}/images/05.png){: .align-center}

This hardware profile must be enabled prior to configure an exporter-map with version ipfix option 315 on a specific line card.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/63x/b-netflow-cg-ncs5500-63x/b-netflow-cg-ncs5500-63x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/63x/b-netflow-cg-ncs5500-63x/b-netflow-cg-ncs5500-63x_chapter_010.html)

### netflow fpc-enable

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile netflow ?
  fpc-enable       Netflow full packet capture enable
  ipfix315-enable  IPFIX 315 enable
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile netflow fpc-enable ?
  location  Location of NETFLOW config
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![05.png]({{site.baseurl}}/images/05.png){: .align-center}

This feature introduced in 7.0.1 enables the full packet capture mode, and is documented externally on [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/b-ncs5500-netflow-cli-reference/b-ncs5500-netflow-cli-reference_chapter_01.html#wp4044800456](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/b-ncs5500-netflow-cli-reference/b-ncs5500-netflow-cli-reference_chapter_01.html#wp4044800456)  

Full MPLS+IP headers are displayed only for L3VPN. For L2VPN, the netflow code does not support any decoding from CW onwards (a XR netflow limitation, and not specific to NCS5500). The Netflow Full Packet Capture is required to capture non-IPoMPLS packets.  

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/70x/configuration/guide/b-netflow-cg-ncs5500-70x/b-netflow-cg-ncs5500-70x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/70x/configuration/guide/b-netflow-cg-ncs5500-70x/b-netflow-cg-ncs5500-70x_chapter_010.html)

### offload

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile offload ?
  1  BFDv6 and Bsync
  2  BFDv6 and Route download
  3  Route download and Bsync
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

This profile, introduced in 6.6.1, enables the hardware offload of the IPv6 BFD for NCS5501-SE. 
Bsync here refers to the PTP (timing feature) capability.
On NCS5501-SE, we can use two cores and we need to select between 3 options: Bsync, BFDv6 and the acceleration of the route download in the NL12k eTCAM.
The problem doesn't exist on other Jericho + NL12k eTCAM systems since they don't support timing and doesn't exist either on the Jericho+ systems since they use an OP eTCAM.  
Option 1 is used by default. Not relevant for other platforms than NCS5501-SE.  

Doesn't work for Segment Routing (same as IPv4) and doesn't support Explicit NULL. Only fixed values of timers can be used (3.3 msec, 10 msec, 100 msec, 1 sec, 10 sec).

_External documentation_:  
- [https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv6-implementation-on-ncs5500-platform/ba-p/3771621](https://community.cisco.com/t5/service-providers-blogs/bfd-over-ipv6-implementation-on-ncs5500-platform/ba-p/3771621)

### qos

Many options behind the qos profiles...

**_hw-module profile qos hqos-enable_**

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos ?
  bvi-l2qos-disable  Disable L2QOS on BVI interfaces
  hqos-enable        Enable Hierarchical QoS
  ingress-model      QoS model for ingress feature
  ipv6               Configure ipv6 protocol
  max-classmap-size  max class map size
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos hqos-enable ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

If you need to apply any kind of QoS policy on a sub-interface, it's mandatory to enable this hardware profile. Even for simple policer in ingress L3 sub-if or shaper in egress L3 sub-if.  
Of course, it's also necessary if you need to apply hierarchical quality of service with parent/children structure.

_Restrictions_  
- HQoS profile and ingress peering profile do not work simultaneously. And hence, features requiring peering profile also do not work with HQoS profile enabled.
- Starting from 6.5.1: Lawful Intercept no longer needs the peering profile and therefor can be used with HQoS profile
- PBTS feature does not work when HQoS profile is enabled.
- A maximum of 896 bundle sub-interfaces are only supported in the system even if there are no QoS policies applied. This is because of internal LAG_ID resource consumption in HQoS profile mode for bundle sub-interfaces with/without QoS policies being applied.
- A maximum of 4 priority levels are only supported in HQoS profile mode unlike the default mode where 7-priority levels are supported. The restriction also applies to physical and bundle main interface policies where 7-level priorities were previously used in non-HQoS profile mode.

_External documentation_:  
[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html#wp2922716190](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html#wp2922716190)

**_hw-module profile qos max-classmap-size_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos ?
  bvi-l2qos-disable  Disable L2QOS on BVI interfaces
  hqos-enable        Enable Hierarchical QoS
  ingress-model      QoS model for ingress feature
  ipv6               Configure ipv6 protocol
  max-classmap-size  max class map size
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos max-classmap-size ?
  16  Max 16 class-maps per policy
  32  Max 32 class-maps per policy
  4   Max 4 class-maps per policy
  8   Max 8 class-maps per policy
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![05.png]({{site.baseurl}}/images/05.png){: .align-center}

The size of the classmap represents the number of match statements it contains. It has a direct impact on the  number of L2 interfaces where you can apply QoS.

By default, class-maps are are configured to contain 32 match statements. With the 4096 available counters, it translates into a maximum of 256 L2 interfaces per NPU, 128 if used on bundle interfaces (with the default 2-counter mode we will detail below).

This hardware profile defines a different class-map size, offering higher scale for L2 interfaces with QoS:

| Class-map Size | Scale on bundle per NPU | Scale per NPU |
|:------:|:------:|:-------:|
| 4 | 1024 | 2000 |
| 8 | 512 | 1024 |
| 16 | 256 | 512 |
| 32 (default) | 128 | 256 |

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp9836538690](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp9836538690)

**_hw-module profile qos ingress-model peering_**

![06.png]({{site.baseurl}}/images/06.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos ?
  bvi-l2qos-disable  Disable L2QOS on BVI interfaces
  hqos-enable        Enable Hierarchical QoS
  ingress-model      QoS model for ingress feature
  ipv6               Configure ipv6 protocol
  max-classmap-size  max class map size
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos ingress-model ?
  default  Default model for ingress QoS
  peering  Peering model for ingress QoS
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile qos ingress-model peering ?
  location  Location of QoS config
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

This profile allows hybrid ACLs to classify traffic and set a specific qos-group to it.  
But instead of the traditional range supported by qos-groups (0 to 7), we can allocate a much larger range from 0 to 511.  
This qos-group can be used as a match condition for the traffic in egress port.  
With this extension from 8 to 512 values, we improve the granularity of the classification.

To disable this feature, use "no hw-module profile qos ingress-model peering" and NOT "hw-module profile qos ingress-model default".  

Note: HQoS profile and ingress peering profile do not work simultaneously: features requiring peering profile like LI also do not work with HQoS profile enabled.
{: .notice--info}

Note2: L2 ACL will not work if qos peering is enabled on the line cards.
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/63x/b-qos-cg-ncs5500-63x/b-qos-cg-ncs5500-63x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/63x/b-qos-cg-ncs5500-63x/b-qos-cg-ncs5500-63x_chapter_010.html)
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp9836538690](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp9836538690)

**_hw-module profile qos ipv6 short_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ?
  hqos-enable        Enable Hierarchical QoS
  ingress-model      QoS model for ingress feature
  ipv6               Configure ipv6 protocol
  max-classmap-size  max class map size
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ipv6 ?
  short               Configure ipv6 source short address tcam lookup
  short-l2qos-enable  Enable l2qos feature which requires to reduce ipv6 dest mask to 96 bits
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ipv6 short ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![04.png]({{site.baseurl}}/images/04.png){: .align-center}

Introduced in IOS XR 6.5.1 for the support of the QPPB feature.  
Only mandatory to apply it (and reload the line card) for IPv6, it's not needed for IPv4 QPPB activation.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/66x/b-qos-cg-ncs5500-66x/b-qos-cg-ncs5500-66x_chapter_010.html#id_77442](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/66x/b-qos-cg-ncs5500-66x/b-qos-cg-ncs5500-66x_chapter_010.html#id_77442)

**_hw-module profile qos ipv6 short-l2qos-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ?
  hqos-enable        Enable Hierarchical QoS
  ingress-model      QoS model for ingress feature
  ipv6               Configure ipv6 protocol
  max-classmap-size  max class map size
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ipv6 ?
  short               Configure ipv6 source short address tcam lookup
  short-l2qos-enable  Enable l2qos feature which requires to reduce ipv6 dest mask to 96 bits
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile qos ipv6 short-l2qos-enable ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![04.png]({{site.baseurl}}/images/04.png){: .align-center}

From CCO: "To enable classification of IPv6 packets based on (CoS, DEI) on L3 sub-interfaces, run the hw-module profile qos ipv6 short-l2qos-enable command in the XR Config mode."  
This profile, introduced in 7.1.1, replaces the "short" one and extends the support of the classification to the COS/DEI fields in the IPv6 headers, reducing the destination address to 96bits.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1849861368](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1849861368)

### segment-routing srv6 

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile segment-routing ?
  srv6  Configure support for SRv6 and its paramaters
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile segment-routing srv6 ?
  encapsulation  Configure encapsulation parameters
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile segment-routing srv6 encapsulation ?
  traffic-class  Control traffic-class field on IPv6 header
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile segment-routing srv6 encapsulation traffic-class ?
  -0x0-0xff-  Traffic-class value (specified as 2 hexadecimal nibbles)
  propagate   Propagate traffic-class from incoming packet/frame
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile segment-routing srv6 encapsulation traffic-class propagate ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

This profile is required to enable Segment Routing v6. We can define a static TC value or copy it from the payload.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/66x/b-segment-routing-cg-ncs5500-66x/b-segment-routing-cg-ncs5500-66x_chapter_011.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/66x/b-segment-routing-cg-ncs5500-66x/b-segment-routing-cg-ncs5500-66x_chapter_011.html)

### sr-policy

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile sr-policy ?
  V6-Null-label-autopush  Configure IPV6 NULL label autopush for SR policy
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile sr-policy V6-Null-label-autopush ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

This profiles enables the V6 null label autopush over SR-policy. DSCP preserve would be disabled.  
This profile is not supported with 6VPE (v6 null label would be pushed rather than 6vpe label).  
With this feature, we can use up to 12 labels for v6 too.

### stats

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats ?
  acl-permit         Enable ACL permit stats.
  enh-sr-policy      Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr         Enable ingress SR stats profile counter.
  qos-enhanced       Enable enhanced QoS stats.
  tx-scale-enhanced  Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

Enabling one of these profiles will disable any other previously applied.

### stats acl-permit

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats ?
  acl-permit         Enable ACL permit stats.
  enh-sr-policy      Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr         Enable ingress SR stats profile counter.
  qos-enhanced       Enable enhanced QoS stats.
  tx-scale-enhanced  Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats acl-permit ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

By default, access-lists don't count permit ACE hits but only the deny ones.  
It could be very important to track the packets permitted, for example when using ABF (ACL Based Forwarding).  
This profile allocates statistic entries to permit ACEs.

If acl-permit is configured, qos-enhanced or other options are disabled. To return to the default mode, use "no hw-module..." and reload.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/b-ip-addresses-cr-ncs5500/b-ncs5500-ip-addresses-cli-reference_chapter_01.html#id_82511](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/b-ip-addresses-cr-ncs5500/b-ncs5500-ip-addresses-cli-reference_chapter_01.html#id_82511)
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html)
- [https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/)

### stats ingress-sr

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats ?
  acl-permit         Enable ACL permit stats.
  enh-sr-policy      Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr         Enable ingress SR stats profile counter.
  qos-enhanced       Enable enhanced QoS stats.
  tx-scale-enhanced  Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats ingress-sr ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

Profile enabling per-label statistics at "ingress" for Segment Routing labels (only for the labels within configured SRGB and SRLB).
Once activated, QoS Stats will not work for the same labeled packets.

### stats enh-sr-policy

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile stats ?
  acl-permit         Enable ACL permit stats.
  egress-stats-scale  Enable Egress Stats_Scale profile counter.
  enh-sr-policy      Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr         Enable ingress SR stats profile counter.
  qos-enhanced       Enable enhanced QoS stats.
  tx-scale-enhanced  Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile stats enh-sr-policy ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

Profile increasing the counters available in the egress pipeline from 16K (default) to 24K. These counters are taken from the ingress pipeline, impacting the scale for ACL/QOS/LPTS/etc.   
This command also enables ingress SR counters.  
Higher egress stats: upto 16K stats available for MPLS cases like RSVP-TE and SR-TE, 4K for ARP/ND and 4K for L2-AC's. 

### stats qos-enhanced

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats ?
  acl-permit         Enable ACL permit stats.
  enh-sr-policy      Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr         Enable ingress SR stats profile counter.
  qos-enhanced       Enable enhanced QoS stats.
  tx-scale-enhanced  Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats qos-enhanced ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

By default, we only use two counters for policers.  
This profile enables the 4-counter mode:
- In 2-counter mode, statistics for “confirm” and “violate” packets are collected in hardware and displayed to the user via the “show policy-map” command
<div class="highlighter-rouge">
<pre class="highlight">
<code>Class C1
  Classification statistics          (packets/bytes)     (rate - kbps)
    Matched             :            52198655/6681427840           13
    Transmitted         :             6138929/785782912            1
    Total Dropped       :            46059726/5895644928           12
  Policing statistics                (packets/bytes)     (rate - kbps)
    Policed(conform)    :             6138929/785782912            1
    Policed(exceed)     :                      <mark>0/0</mark>                      0
    Policed(violate)    :            46059726/5895644928           12
    Policed and dropped :            46059726/5895644928     </code>
</pre>
</div> 
- In 4-counter mode, additionally “exceed” packets statistics are collected and displayed. 
<div class="highlighter-rouge">
<pre class="highlight">
<code>Class C1
  Classification statistics          (packets/bytes)     (rate - kbps)
    Matched             :            52198655/6681427840           13
    Transmitted         :             6138929/785782912            1
    Total Dropped       :            46059726/5895644928           12
  Policing statistics                (packets/bytes)     (rate - kbps)
    Policed(conform)    :             6138929/785782912            1
    Policed(exceed)     :                <mark>11326/1449728</mark>              0
    Policed(violate)    :            46059726/5895644928           12
    Policed and dropped :            46071052/5897095778    </code>
</pre>
</div>

The trade-off for such visibility will be half the scale.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_01.html#wp9836538690](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_01.html#wp9836538690)

### tx-scale-enhanced

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats tx-scale-enhanced ?
  acl-permit    Enable ACL permit stats.
  ingress-sr    Enable ingress SR stats.
  qos-enhanced  Enable enhanced QoS stats.
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats tx-scale-enhanced acl-permit ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats tx-scale-enhanced ingress-sr ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile stats tx-scale-enhanced qos-enhanced ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![09.png]({{site.baseurl}}/images/09.png){: .align-center}

This profile changes the "stats" EEDB bank to use "half" entries and thereby increases the EEDB scale available. It has been specifically created for large TE deployments, extending the head-end scale from 4k to 14k. Only on J+ systems.

Don't use in mixed (J/J+) chassis. Also it can't support L2 features.

### stats egress-stats-scale

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile stats ?
  acl-permit          Enable ACL permit stats.
  egress-stats-scale  Enable Egress Stats_Scale profile counter.
  enh-sr-policy       Enable Enhanced_SR_Policy_Scale stats profile counter.
  ingress-sr          Enable ingress SR stats profile counter.
  qos-enhanced        Enable enhanced QoS stats.
  tx-scale-enhanced   Enable enhanced TX stats scale (Non L2 stats).
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module profile stats egress-stats-scale ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

Very similar the enh profile, extending the MPLS scale from 8k to 20k counters (4k for ARP/ND) and at the expense of the L2 counters (no L2 with this one).

### tcam acl-prefix

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam ?
  acl-prefix  ACL table to configure
  fib         Forwarding table to configure
  format      format of the tcam entry
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam acl-prefix ?
  percent  percent to configure
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam acl-prefix percent ?
  -0-100-  value in percent
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![07.png]({{site.baseurl}}/images/07.png){: .align-center}

Hybrid (or Scaled) ACLs are stored in two places: iTCAM and eTCAM. But by default in 6.3.2 (onwards), the eTCAM is not carved to receive ACL information.  
This hardware profiles helps with this carving.  

First, it's only relevant for products with eTCAM. Hybrid ACLs are not supported on "base" systems or LC, only on "scale / -SE" ones.  
Second, it's only relevant to products based on Jericho. With Jericho+, the eTCAM is much larger and a portion of the database is allocated to ACLs dynamically, no configuration is needed.  
Finally, pay specific attention to the release used on the router. Indeed, between 6.2.x and before 6.3.2, the eTCAM was carved by default with 20% for ACL and 80% for IPv4 routes. Its not the case before 6.2 and starting from 6.3.2, where default config is allocating 100% of the eTCAM to IPv4 routes.

Note: if you don't configure space for ACL in eTCAM, the hybrid ACL configuration will be rejected at commit.
{: .notice--info}

_External documentation_:  
These subtleties and many more details on Hybrid ACLs are covered in this external blog post:  
[https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/)

### tcam fib

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam ?
  acl-prefix  ACL table to configure
  fib         Forwarding table to configure
  format      format of the tcam entry
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ?
  ipv4     Configure ipv4 addresses in TCAM
  ipv6     Configure ipv6 addresses in TCAM
  v6mcast  Multicast address
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 ?
  unicast  Unicast address
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast ?
  percent  percent to configure
  prefix   ip prefix length
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast percent ?
  -1-100-  value in percent
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast percent 20 ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast prefix ?
  -0-32-  IPv4 prefix length.
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast prefix 24 ?
  percent  prefix percentage to configure
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam fib ipv4 unicast prefix 24 percent ?
  WORD  prefix precent 0.0-100.00000
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![07.png]({{site.baseurl}}/images/07.png){: .align-center}

Only valid on the first generation of eTCAM (with Jericho ASIC). These hardware profile CLI permits the allocation / carving of the memory for specific route type. It does influence the way routes are stored in the different databases too.  
It's not operational on J+ eTCAM since the allocation is dynamic and all routes are stored in external TCAM by default.

_External documentation_:  
- Mentioned in the CiscoLive breakout Session: [https://www.ciscolive.com/c/dam/r/ciscolive/emea/docs/2019/pdf/BRKSPG-2900.pdf](https://www.ciscolive.com/c/dam/r/ciscolive/emea/docs/2019/pdf/BRKSPG-2900.pdf)

### tcam format acl

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam ?
  acl-prefix  ACL table to configure
  fib         Forwarding table to configure
  format      format of the tcam entry
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam format ?
  access-list  Access List format
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam format access-list ?
  ipv4  IPv4
  ipv6  IPv6
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile tcam format access-list ipv4 ?
  common-acl       enable common-acl, 1 bit qualifier
  dst-addr         destination address, 32 bit qualifier
  dst-port         destination L4 Port, 16 bit qualifier
  enable-capture   Enable ACL based mirroring (Included by default)
  enable-set-ttl   Enable Setting TTL field (Included by default)
  frag-bit         fragment-bit, 1 bit qualifier
  interface-based  Enable non-shared interface based ACL
  location         Location of format access-list ipv4 config
  packet-length    packet length, 16 bit qualifier
  port-range       ipv4 port range qualifier, 24 bit qualifier
  precedence       precedence/dscp, 8 bit qualifier
  proto            protocol type, 8 bit qualifier
  src-addr         source address, 32 bit qualifier
  src-port         source L4 port, 16 bit qualifier
  tcp-flags        tcp-flags, 6 bit qualifier
  ttl-match        Enable matching on TTL field
  udf1             user defined filter
  udf2             user defined filter
  udf3             user defined filter
  udf4             user defined filter
  udf5             user defined filter
  udf6             user defined filter
  udf7             user defined filter
  udf8             user defined filter
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![05.png]({{site.baseurl}}/images/05.png){: .align-center}

These hardware profiles are necessary to enable the UDK/UDF (User-Defined TCAM Key and Field) used with ingress ACL.  
- They permit the definition of the TCAM key.
- Allow For ttl rewrite based on Tunnel policy if used with set-ttl and match ttl
- Disable ACL sharing and support many small and unique ACL if used with interface-based

Indeed, the default behavior of an ingress access-list is to be re-usable: that means ACE lines of an ingress ACL applied on multiple ports of the same NPU are only counted twice (it's not the case with egress ACLs).  
This hardware profile when used with the keyword "interface-based" will disable the re-usability and move to the unique-ACL mode.

An example for configuring unique ACL for both IPv4 and IPv6 with the following fields (SRC_IP, DST_IP, SRC_PORT, DST_PORT) available for ACE matching is:  

<div class="highlighter-rouge">
<pre class="highlight">
<code>hw-module profile tcam format access-list ipv4 src-addr src-port dst-addr dst-port interface-based
hw-module profile tcam format access-list ipv6 src-addr dst-addr dst-port interface-based</code>
</pre>
</div>

Note: switching functionality between shared and interface-based ACLs, all existing interface attachments needs to be removed.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/63x/b-ip-addresses-configuration-guide-ncs5500-63x/b-ip-addresses-configuration-guide-ncs5500-63x_chapter_010.html)

## quad

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:55A2-MOD-SE(config)#hw-module quad 0 location 0/0/CPU0 mode ?
  WORD  10g or 25g, (10g mode also operates 1g transceivers)
RP/0/RP0/CPU0:55A2-MOD-SE(config)#</code>
</pre>
</div>

Feature used only on 25GE-capable platforms (25GE native, not related to the breakout option here):  
- all NCS55A2-MOD versions
- NCS55A1-24Q6H-S
- NCS55A1-48Q6H  
By default, those 1G/10G/25G ports are configured in TF mode only. An SFP or SFP+ optic inserted will not turn up. The profile allows the configuration of a block of 4 ports (a quad) to be used for 1G/10G mode.  
What quad 0 represents will depend on each platform.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/routers/ncs5500/software/interfaces/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-66x/b-interfaces-hardware-component-cg-ncs5500-66x_chapter_01100.html](https://www.cisco.com/c/en/us/td/docs/routers/ncs5500/software/interfaces/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-66x/b-interfaces-hardware-component-cg-ncs5500-66x_chapter_01100.html)

## service

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module service ?
  offline  Take all services on the card offline
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module service offline ?
  location  Location to configure
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module service offline location ?
  0/0/CPU0    Fully qualified location specification
  0/3/CPU0    Fully qualified location specification
  0/4/CPU0    Fully qualified location specification
  0/6/2       Fully qualified location specification
  0/6/CPU0    Fully qualified location specification
  0/7/CPU0    Fully qualified location specification
  0/RP0/CPU0  Fully qualified location specification
  WORD        Fully qualified location specification
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![05.png]({{site.baseurl}}/images/05.png){: .align-center}

Meant to put a line card offline, but according to the following release notes:  
"The offline diagnostics functionality is not supported in NCS 5500 platform. Therefore, the hw-module service offline location command will not work. However, you can use the (sysadmin)# hw-module shutdown location command to bring down the LC."  

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/66x/release/notes/b-release-notes-ncs55k-r663.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/66x/release/notes/b-release-notes-ncs55k-r663.html)

## tcam

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module tcam ?
  fib  Forwarding table to configure
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module tcam fib ?
  ipv4  Configure ipv4 protocol
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module tcam fib ipv4 ?
  scaledisable  Configure scale mode for TCAM card
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module tcam fib ipv4 scaledisable ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>

![07.png]({{site.baseurl}}/images/07.png){: .align-center}

This profile exists specifically for the application of URPF on a Jericho-based eTCAM system or line card.  
It will disable a mechanism named "double capacity" that allowed to double the size of the 80b key memory, extending the route count from 1M to 2M entries.  

_External documentation_:  
Details are available on the external blog post:  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/](https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/)  
On CCO:  
[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/62x/b-system-security-cg-ncs5500-62x/b-system-security-cg-ncs5500-62x_chapter_01001.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/62x/b-system-security-cg-ncs5500-62x/b-system-security-cg-ncs5500-62x_chapter_01001.html)

## route-stats 

In IOS XR 7.0.2:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-702(config)#hw-module route-stats ?
  l3mcast  Layer 3 Multicast
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module route-stats l3mcast ?
  ipv4  IPv4 Multicast
  vrf   Vrf Name
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module route-stats l3mcast ipv4 ?
  res   Access list name - maximum 64 characters
  WORD  Access list name - maximum 64 characters
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module route-stats l3mcast ipv4 TEST ?
  -cr-
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
</pre>
</div>
  
In IOS XR 6.6.3:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats ?
  l3mcast  Layer 3 Multicast
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ?
  ipv4  IPv4 Multicast
  ipv6  IPv6 Multicast
  vrf   Vrf Name
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv4 ?
  egress   per route per OIF stats
  ingress  per route stats
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv4 ingress ?
  WORD             Access list name - maximum 64 characters
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv4 egress ?
  WORD             Access list name - maximum 64 characters
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv6 ?
  egress   per route per OIF stats
  ingress  per route stats
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv6 ingress ?
  WORD  Access list name - maximum 64 characters
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast ipv6 egress ?
  WORD  Access list name - maximum 64 characters
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast vrf ?
  WORD     Name of VRF
  default  Default VRF
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module route-stats l3mcast vrf foo ?
  ipv4  IPv4 Multicast
  ipv6  IPv6 Multicast
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

Where mcast-counter is an access-list defined in the configuration

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP0/0/RP0/CPU0:NCS5500-663(config)# ipv4 access-list mcast-counter
RP0/0/RP0/CPU0:NCS5500-663(config-acl)# 10 permit ipv4 host 10.1.1.2 host 224.2.151.1
RP0/0/RP0/CPU0:NCS5500-663(config-acl)# 20 permit ipv4 10.1.1.0/24 232.0.4.0/22
RP0/0/RP0/CPU0:NCS5500-663(config-acl)#commit</code>
</pre>
</div>

Before the introduction of this feature, we only have a very brief implementation of an hw-profile only in 6.1.31 ("hw-module profile mfib stats"). Otherwise the count packets and bytes per (S,G) was not available. 

The ingress stats are always per (S,G). The egress stats are always per OLE.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/multicast/66x/configuration/guide/b-multicast-cg-ncs55k-66x/b-multicast-cg-ncs55k-66x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/multicast/66x/configuration/guide/b-multicast-cg-ncs55k-66x/b-multicast-cg-ncs55k-66x_chapter_010.html)
 
## stats-fpga

<div class="highlighter-rouge">
<pre class="highlight">
<code>hw-module stats-fpga location 0/x/CPU0 enable</code>
</pre>
</div>

Currently only supported on the NCS560. In the roadmap for other platforms equiped with this FGPA:
- MOD-SE Line Cards
- NCS55A2-MOD-SE

## vrrpscale

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5500-663(config)#hw-module vrrpscale ?
  enable  enable VRRP scaling
RP/0/RP0/CPU0:NCS5500-663(config)#hw-module vrrpscale enable ?
  -cr-
RP/0/RP0/CPU0:NCS5500-663(config)#</code>
</pre>
</div>

Extends the scale from 16 (or even 13 with BFD) to 255.  
In 7.1.1, it's also required to enable HSRP.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/b-ip-addresses-cr-ncs5500/b-ncs5500-ip-addresses-cli-reference_chapter_01001.html#id_91192](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/b-ip-addresses-cr-ncs5500/b-ncs5500-ip-addresses-cli-reference_chapter_01001.html#id_91192)

## Conclusion

Multiple hardware profiles have been added release after release to meet certain use-cases. This document helped at clarifying all the variations available.  

Note that several profiles are orthogonal and can not be used simultaneously, like:

|   |   |
|:-----:|:-----:|
| hw-module profile stats acl-permit | hw-module profile stats qos-enhanced |
| hw-module profile qos ingress-model peering loc | QPPB features |
| hw-module profile qos ingress-model peering loc | hw-module profile qos hqos-enable |
| hw-module profile qos hqos-enable | PBTS |
| hw-module fib mpls label lsr-optimized | hw-module fib ipv4 scale internet-optimized |
| hw-module fib mpls ldp lsr-optimized | No EVPN Services |

Bookmark this link and check regularly for updates.