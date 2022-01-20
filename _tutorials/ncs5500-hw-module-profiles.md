---
published: true
date: '2020-02-14 16:39 +0100'
title: NCS5500 Hw-module Profiles
author: Nicolas Fevrier
excerpt: WIP document describing the various options behind the NCS5500 hw-module CLI
tags:
  - iosxr
  - dnx
position: top
---
{% include toc icon="table" title="NCS5500 Hw Profiles" %} 

|   |   |
|:-----|:-----|
| 2020-Feb-14 | Document Creation |
| 2020-Mar-12 | Correction of the max-classmap-size description |
| 2020-Apr-08 | Correction: HQoS not needed for ingress policer on sub-if |
| 2020-Jul-31 | Add: hw-mod profile bw-threshold |
| 2020-Aug-10 | Add: comment on the need to use UDK profile for packet-length match in ACL |
| 2021-Apr-3  | Add: hw-mod profiles till 7.3.1, updated the support for J2 based platforms for exisiting profiles | 
| 2021-Sept-9  | Add: hw-mod profiles till 7.4.1, updated the support for J2 based platforms for exisiting profiles | 
| 2022-Jan-20  | Add: hw-mod profiles till 7.5.1  | 




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
- Riadh Habibi
- Richard Poll
- Tejas Lad
- Angu Chakravarthy
- Paban Sarma

## Hardware Module CLI hierarchy

For this article, we use IOS XR 6.6.3, 7.0.2, 7.3.1, 7.4.1 . The document will be updated regularly.  

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

![Screenshot 2021-04-03 at 12.57.20 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.57.20 PM.png)


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

Note1: Please pay attention to the form of the command since it could lead to confusions: for IPv4 it's internet-optimized while it's internet-optimized**-disable** for IPv6.
{: .notice--info}

Note2: This profiles are not applicable for J2 based systems.
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

Note1: in an IGP domain, only the router's loopbacks need to be bound to labels. It's a best practise to use filters to reduce the IGP to MPLS relationship.
{: .notice--info}

Note2: This profiles is not applicable for J2 based systems.
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

Note1: lsr-optimized mode was introduced in 6.5.x and was not supported on J+ w/ eTCAM systems initially, this limitation is removed starting from 7.1.1.
{: .notice--info}

Note2: this profile can not be used with the internet-optimized.
{: .notice--info}

Note3: This profiles are not applicable for J2 based systems.
{: .notice--info}

### recycle

**_hw-module fib recycle service-over-rsvpte_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib recycle ?
  service-over-rsvpte  Recycle traffic for BGP services going over RSVP TE
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib recycle service-over-rsvpte ?
  -cr-  
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib recycle service-over-rsvpte 
Wed Mar 31 23:10:12.063 PDT
In order to activate/deactivate recycling traffic for BGP services going over RSVP TE, you must manually reload the chassis/all line cards
RP/0/RP1/CPU0:5508-1-731(config)#
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.58.02 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.58.02 AM.png)

To enable L3 BGP services over NCS5500/5700 use hw-module profile hw-module fib recycle service-over-rsvpte”. Prior to this feature, BGP services (6PE, VPNv4) were not supported over LDPoRSVPTE on NCS5500. With introduction of this feature, in IOS-XR 7.3.1 on systems with J/J+ and IOS-XR 7.4.1 on J2-comp-mode this use cases is possible. The is achieved via recycle approach.  In first pass,  IGP local label and BGP label is added and then packet is recycled. Recycled packet is treated as regular IGP MPLSoRSVPTE packet in second pass. In second pass, IGP local label is swapped with IGP out label and then pushed RSVP TE label and packet is sent out. In order to activate/deactivate recycling traffic for BGP services going over RSVP TE, you must manually reload the chassis/all line cards

Note: Enabling this profile, packets meant for the TE tunnel gets recycled
EVPN over LDPoTE is supported only on J/J+. 
{: .notice--info}

## bgp-pic multipath

**_hw-module fib bgp-pic multipath-core enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib bgp-pic multipath-core ?
  enable  Enable pic core in forwarding chain
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib bgp-pic multipath-core enable ?
  -cr-  
RP/0/RP1/CPU0:5508-1-731(config)#hw-module fib bgp-pic multipath-core enable 
Thu Apr  1 02:49:14.402 PDT
In order to activate/deactivate bgp multipath pic core, you must manually reload the chassis/all line cards
RP/0/RP1/CPU0:5508-1-731(config)#
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.59.58 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.59.58 AM.png)

To avoid BGP prefix dependent convergence, when multiple ECMP paths are available via IGP, use the hw-module profile fib multipath-core enable. This helps in achieving sub second convergence. The BGP protocol will select more than 1 primary path as primary path set and 1 backup path for primary path set. It will install the primary and backup path pair in the RIB. FIB will replicate same backup path for all primary paths and program all path as a protected path. To disable the feature, need to use below CLI. After disabling the CLI router reload is recommended.“no hw-module fib bgp-pic multipath-core enable”. 

Note: In order to activate/deactivate bgp multipath pic core, you must manually reload the chassis/all line cards.BGP LU level 1 is not supported. L3VPN/6PE/6VPE pic core multipath is not supported over igp
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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)
  

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
<code>RP/0/RP0/CPU0:NCS5500-702(config)#do sh int brief | i "(0/3/0/28|0/3/0/29)"
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
RP/0/RP0/CPU0:NCS5500-702(config)#</code>
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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

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

Note: This profiles is not applicable for J2 based systems
{: .notice--info} 


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

Note: Can be enabled and disabled without requiring a system reload. This profile is not needed on J2 based systems.
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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

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


### bundle-hash


**_hw-module profile bundle-hash ignore-ingress-port_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash ignore-ingress-port ?  
  -cr-  
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash ignore-ingress-port
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.37.35 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.37.35 AM.png)

This profile was introduced in IOS-XR 7.1.2. After enabling this profile ingress traffic port is removed from the hash-key computation. This results in different hash-key value gets computation. This alters the traffic distribution across the bundle members. This is a global CLI and applies to all Line cards and all bundles on the chassis. This profile doesn’t require a chassis or a LC reload.


**_hw-module profile bundle-hash per-packet-round-robin_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash ?
  hash-index              configure which polynomial to use in bundle-hash
  ignore-ingress-port     Disable ingress port during bundle hash computation
  per-packet-round-robin  Enable per-packet round robin loadbalancing for all bundles in system
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash 
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash per-packet-round-robin ?
  -cr-  
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.40.07 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.40.07 AM.png)

This profile is introduced in IOS XR 7.3.1. Enabling this profile, bundles will start egressing traffic in a per-packet round robin manner across all members. This suppresses any internal load balancing algorithm and it becomes purely round robin. This is a global CLI and applies to all bundles across all locations/LCs. 

Note: This profile doesn’t require a chassis or a LC reload.
When per packet round robin mode is enabled, all bundle links will be equally used for egress. Hence, the bundle-hash CLI tool or ‘show cef exact-route’ command to predict the egress member link will not be of any use.  
{: .notice--info}


**_hw-module profile bundle-hash hash-index <> location <>_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash hash-index ?           
  1   Use Polynomial value 0x8011
  10  Use LB-Key-Pkt-Data directly
  11  Use counter incremented every packet
  12  Use counter incremented every two clocks
  2   Use Polynomial value 0x8423
  3   Use Polynomial value 0x8101
  4   Use Polynomial value 0x84A1
  5   Use Polynomial value 0x9019
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash hash-index 10 ?
  location  Location of bundle-hash polynomial config
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash hash-index 10 location ?
  0/0/CPU0  Location of bundle-hash polynomial config
  0/3/CPU0  Location of bundle-hash polynomial config
  0/7/CPU0  Location of bundle-hash polynomial config
  WORD      Location of bundle-hash polynomial config
RP/0/RP1/CPU0:5508-1-731(config)#hw-module profile bundle-hash hash-index 10 location
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.41.57 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.41.57 AM.png)

This is the config to adjust the LAG load balancing algorithm by changing the hash polynomial used by ASIC. The router continues to use the existing 7-Tuples algorithm, but user can change the polynomial value used internally. This results in different load-balancing which may be better in some cases. This profile is per location or LC based. So the change is applicable only for traffic streams ingressing on that particular Line Card. Hash-key is calculated based on ingress LC. Hash-key value decides which bundle member will be selected for egress, irrespective of whichever LC bundle member belongs to. User can configure same hash-index on all LCs if desired.


Note: Any other hash-index is not valid or there are some internal limitations. Hence only CLI should be used to modify the hash-index and not via shell.
Hash-indices 11 and 12 result in a per-pkt based distribution across bundle. So should be used with caution/after testing. This profile doesn’t require a chassis or a LC reload. When there are multiple NPU’s/ASIC’s on a LC, and hash index is configured on that location, that configuration gets applied to each NPU.
{: .notice--info}




### bw-threshold

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-2-702(config)#hw-module profile bw-threshold ?
  WORD  value in percent: 0-100,in increments of 10
RP/0/RP0/CPU0:5508-2-702(config)#hw-module profile bw-threshold 20 ?
  cr
RP/0/RP0/CPU0:5508-2-702(config)</code>
</pre>
</div>

![02.png]({{site.baseurl}}/images/02.png){: .align-center}

This feature allows the NPU to monitor of the number of fabric interfaces "up".  
When the fabric bandwidth falls below the configured bandwidth threshold on any one ASIC, the front panel interfaces on the line card are forced down, triggering a network re-convergence (through routing protocols). If we cross the threshold in the other direction, the front panel interfaces are brought back up again.  
The "bring down threshold" is kept 2 fabric links per ASIC below the "bring up threshold".

This algorithm does not apply to fixed systems where the fabric links are not connected.  

Check "Set Fabric Bandwidth Threshold" in the installation guide:  
[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/hardware-install/b-ncs5500-hardware-installation-guide/b-ncs5500-hardware-installation-guide_chapter_011.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/hardware-install/b-ncs5500-hardware-installation-guide/b-ncs5500-hardware-installation-guide_chapter_011.html)

Note: This profile is not needed for J2 based systems.
{: .notice--info}

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

![Screenshot 2021-04-03 at 2.31.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 2.31.43 PM.png)


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

There are 6 modes which can be configured. The default (i.e. with no configuration) selects the algorithm which has been running in previous releases.
- ip-tunnel allows the hashing algorithm to use the outer IPv4 GRE header even when doing an IP tunnel decapsulation. 
- layer2 allows the hashing algorithm to be able to use the inner IP header information when doing layer 2 forwarding and the inner payload is MPLS.
- gtp allows hashing based upon the tunnel id in GTP-U (UDP DST 2152) packets. (6.5.1 release or later). When this option is selected, the hashing is based on ip[v4/v6] (src, dest, protocol) + udp (src, dest) + tunnel endpoint id).
- gtp-mpls allows hashing based upon the tunnel id in GTP-U (UDP DST 2152) packets. (7.2.1 release or later). When this option is selected, the hashing is based on tunnel endpoint id.
- mpls-safe-speculative-parsing mode was introduced (6.5.3 release or later) for cases where the network has MPLS carrying L2VPN packets without control words. In this case, if the first nibble of the MAC DA address is a 4 or 6, the rest of the packet could be interpreted incorrectly as IPv4 or IPv6 header. In this mode, the algorithm uses a subset of the IPv4 and IPv6 header for load balancing. It does this for MPLS 1-6 labels for IPv6 and MPLS 4-6 labels for IPv4. For MPLS 1-3 labels and IPv4, the hardware can correctly determine whether it’s an IPv4 header or not (uses the checksum).
- L3-only will not use any L4 headers for the hashing. Only IPv4 and IPv6 will be used. It could be useful in certain corner-cases, particularly to guarantee the fragmented packets will take the same path than the first packet of the fragment.
 
These profiles can be activated without requiring a reload of the system or the line card.
{: .notice--info}

**_hw-module profile load-balance algorithm PPPoE_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile load-balance algorithm ?
  L3-only                        L3 Header only Hash.
  PPPoE                          PPPoE session based optimized hash. Reload is required for this option
  gtp                            GTP optimized.
  gtp-mpls                       GTP over MPLS optimized hash.
  ip-tunnel                      IP tunnel optimized.
  layer2                         Layer 2 optimized.
  mpls-safe-speculative-parsing  MPLS safe Speculative parsing.
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile load-balance algorithm PPPoE
Thu Aug 26 11:12:31.971 PDT
reload of all chassis/all line cards is required only for PPPoE option configuration/removal
</code>
</pre>
</div>

![J-Jp-only-Global.png]({{site.baseurl}}/images/J-Jp-only-Global.png)

This new load balancing algorthim is introduced in IOS XR 7.4.1 and currently applicable to only J and J+ based NCS 5500 PIDs. When this hw-module profile is enabled it allows ECMP and LAG hashing based on inner header (IPv4/IPv6) soon after PPPoE header for Layer 2  P2P and P2MP bridging cases. A unique FAT label is allocated for this load-balancing to happen. 

`NOTE: Unlike the other load-balancing profiles, the router needs to be reloaded to enable this mode`

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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

This feature introduced in 7.0.1 enables the full packet capture mode, and is documented externally on [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/b-ncs5500-netflow-cli-reference/b-ncs5500-netflow-cli-reference_chapter_01.html#wp4044800456](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/b-ncs5500-netflow-cli-reference/b-ncs5500-netflow-cli-reference_chapter_01.html#wp4044800456)  

Full MPLS+IP headers are displayed only for L3VPN. For L2VPN, the netflow code does not support any decoding from CW onwards (a XR netflow limitation, and not specific to NCS5500). The Netflow Full Packet Capture is required to capture non-IPoMPLS packets.  

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/70x/configuration/guide/b-netflow-cg-ncs5500-70x/b-netflow-cg-ncs5500-70x_chapter_010.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/netflow/70x/configuration/guide/b-netflow-cg-ncs5500-70x/b-netflow-cg-ncs5500-70x_chapter_010.html)

### offload

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NCS5500-702(config)#hw-module profile offload ?
  1  BFDv6 and Bsync
  2  BFDv6 and Route download
  3  Route download and Bsync
RP/0/RP0/CPU0:NCS5500-702(config)#
</code>
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

Note: This profile is not needed for J2 based systems
{: .notice--info}

### oam

**_hw-module profile oam 48byte-cfm-maid-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile oam ?
  48byte-cfm-maid-enable  Enable 48byte cfm maid feature
  sat-enable              enable SAT feature
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile oam 48byte-cfm-maid-enable ?
  -cr-
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile oam 48byte-cfm-maid-enable
Thu Jan 20 09:55:51.661 UTC
In order to make the oam profile take effect, the router must be manually reloaded.
</code>
</pre>
</div>

![Screenshot 2022-01-20 at 3.40.30 PM.png]({{site.baseurl}}/images/Screenshot 2022-01-20 at 3.40.30 PM.png)

The NCS 5500 and NCS 5700 system has limited format support for MAID/MDID formats for hw-offloaded CFM session (<1 minutes). With introduction to this new hw-module profile, NCS 5700 fixed systems and modular boxes running in J2 native mode will support flexible format MDID/MAIDs for hardware offloaded MEPs. Activation of this mode needs the router to be reloated. 


### qos

Many options behind the qos profiles...

**_hw-module profile qos hqos-enable_**

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

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

If you need to apply any kind of QoS policy on a sub-interface, it's mandatory to enable this hardware profile. Even for simple shaper in egress L3 sub-if (note: you don't need it for ingress shaper).  
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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

The max-classmap-size represents the maximum number of class-map we can use in policy-map. It has a direct impact on the  number of L2 interfaces where you can apply QoS.

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

![Screenshot 2021-04-03 at 12.39.43 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.39.43 PM.png)

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

![]({{site.baseurl}}/images/Screenshot%202021-04-03%20at%2011.05.27%20AM.png)

From CCO: "To enable classification of IPv6 packets based on (CoS, DEI) on L3 sub-interfaces, run the hw-module profile qos ipv6 short-l2qos-enable command in the XR Config mode."  
This profile, introduced in 7.1.1, replaces the "short" one and extends the support of the classification to the COS/DEI fields in the IPv6 headers, reducing the destination address to 96bits.

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1849861368](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1849861368)

**_hw-module profile qos conform-aware-policer_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos ?
  conform-aware-policer           Configure Conform Aware Policer mode
  hqos-enable                     Enable Hierarchical QoS
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  max-classmap-size               max class map size
  physical-hqos-enable            Enable Hierarchical QoS on physical interfaces
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in t
he same ingress QoS policy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos conform-aware-policer ?
  -cr-  
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos conform-aware-policer
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.05.27 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.05.27 AM.png)

To enable the conform-aware hierarchical policy feature use the _hw-module profile qos conform-aware-policer_ command. It was introduced in release IOS-XR 7.2.1. Implementing the profile conform aware hierarchical policy, allows conform traffic from the child level policy to get priority over Exceed or Violate traffic at parent level policy. Prior to this profile, there was no way that conform traffic belonging to child policy was getting priority over the parent level policy.  When this profile is enabled the entire system is put in color aware mode as compared to color blind mode by default. There is no effect on other features or resources.

Note: In order to activate this new npu profile, you must manually reload the chassis
The support on J2 based platforms is only in native mode from release IOS XR 7.4.1
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0100.html)


**_hw-module profile qos shared-policer-per-class-stats_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos ?
  conform-aware-policer           Configure Conform Aware Policer mode
  hqos-enable                     Enable Hierarchical QoS
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  max-classmap-size               max class map size
  physical-hqos-enable            Enable Hierarchical QoS on physical interfaces
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in t
he same ingress QoS policy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos shared-policer-per-class-stats ?
  -cr-  
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos shared-policer-per-class-stats
</code>
</pre>
</div>

![]({{site.baseurl}}/images/Screenshot%202021-04-03%20at%2011.05.27%20AM.png)

This profile was introduced in IOS-XR 7.2.1. Implementing this profile, will provide an option for token bucket sharing among two or more classes. Prior to this, there was no way to share a policer token bucket among two or more class. This token sharing depends on the incoming traffic rate, so there is no guarantee that both c1 and c2 will get equal share. However, if c2 doesn’t have any traffic, then all the bandwidth will be used by c1. This CLI will need system reload to enable the feature. The stats for each class will be available separately. In Shared Policer feature Per-Class Mode, policer bucket will be shared among two or more classes. It will also allow individual class statistics rather than aggregated statistics. Without enabling this feature as well the shared policer will work with respect to token bucket sharing, however the stats will be available only as aggregate stats, and not per individual class stats.

Note: In order to activate this new npu profile, you must manually reload the chassis
The support on J2 based platforms is only in native mode from release IOS XR 7.4.1
Policy based tunnel selection (PBTS) is disabled when we enable this profile
{: .notice--info}

_External documentation_:  
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_010.html#wp4161007330](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_010.html#wp4161007330)


**_hw-module profile qos wred-stats-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos wred-stats-enable
  -cr-  
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos shared-policer-per-class-stats
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.18.57 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.18.57 AM.png)


WRED-stats accounting feature on NCS5700 platform will be introduced from IOS-XR 7.4.1. WRED stats will be supported on systems based on J2 with OP2 TCAM only in native mode. Statistics like WRED FWD, WRED Drop count will be available for all the discard-class 0,1,2. WRED stats can be used as an indication of network congestion bottleneck and network planning can be done accordingly.

Note:In order to activate this new npu profile, you must manually reload the chassis
The support is only on J2 platforms with e-tcam in native mode from release IOS XR 7.4.1
{: .notice--info}


**_hw-module profile qos qosg-dscp-mark-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos ?
  conform-aware-policer           Configure Conform Aware Policer mode
  hqos-enable                     Enable Hierarchical QoS
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  max-classmap-size               max class map size
  physical-hqos-enable            Enable Hierarchical QoS on physical interfaces
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in t
he same ingress QoS policy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos qosg-dscp-mark-enable 10 ?
  0-63  Second DSCP/Precedence value
  -cr-    
RP/0/RP0/CPU0:IOS(config)#hw-module profile qos qosg-dscp-mark-enable 10
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.22.42 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.22.42 AM.png)

To set the qos-group and DSCP values within the same QoS policy that is applied in the ingress direction, use the hw-module profile qos qosg-dscp-mark-enable command. This profile was introduced in IOS-XR 7.1.2. With the use of this profile, we can configure a single ingress policy with both set dscp/precedence and set qos-group at the same time. We can configure set dscp or prec and set qos-group under the same policy-map but not inside the same class map in a policy map. In order to activate this new npu profile, you must manually reload the chassis

_External documentation_:
- [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1468010117](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/b-ncs5500-qos-cli-reference/b-ncs5500-qos-cli-reference_chapter_0101.html#wp1468010117)


**_hw-module profile qos ecn-marking-stats_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ios#con t
Fri Mar 26 10:35:10.739 UTC
RP/0/RP0/CPU0:ios(config)#hw-module profile qos ?
  conform-aware-policer           Configure Conform Aware Policer mode
  ecn-marking-stats               Enable ECN marking stats mode
  hqos-enable                     Enable Hierarchical QoS
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  max-classmap-size               max class map size
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in the same ingress QoS policy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
  wred-stats-enable               Enable Wred egress stats
RP/0/RP0/CPU0:ios(config)#hw-module profile qos ecn-marking-stats 
Fri Mar 26 10:35:20.004 UTC
In order to activate this profile, you must manually reload the chassis/all line cards
</code>
</pre>
</div>

![Screenshot 2021-04-03 at 11.24.40 AM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 11.24.40 AM.png)

This profile helps to enable the counters for ECN marked packets when the packet is going out of the node. It was introduced in release IOS-XR 7.3.2. 
ECN feature is enabled by default in the system. But counters for ECN marked packets is not enabled due to stats resource limitation. When this hw-module is enabled, we enable rules in hardware to count ECN marked packets during congestion in the system.

In order to activate this new npu profile, you must manually reload the chassis
With this profile enabled, Egress IPv4 and IPv6 ACL will not work due to PMF resource limitation in egress.

Note: In order to activate this new npu profile, you must manually reload the chassis
With this profile enabled, Egress IPv4 and IPv6 ACL will not work due to PMF resource limitation in egress.
{: .notice--info}

**_hw-module profile qos l2-match-dest-addr-v4v6_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile qos ?
  arp-isis-priority-enable        Prioritize ISIS and ARP packets
  conform-aware-policer           Configure Conform Aware Policer mode
  ecn-marking-stats               Enable ECN marking stats mode
  hqos-enable                     Enable Hierarchical QoS on physical and sub-interfaces
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  l2-match-dest-addr-v4v6         Enable l2qos match on ipv4/ipv6 destination address
  max-classmap-size               max class map size
  physical-hqos-enable            Enable Hierarchical QoS only on physical interfaces, and not on sub-interfaces
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in the same ingress QoS po
licy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
  wred-stats-enable               Enable Wred egress stats

RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile qos l2-match-dest-addr-v4v6
Thu Jan 20 09:52:48.156 UTC
In order to activate this profile, you must manually reload the chassis/all line cards
</code>
</pre>
</div>

![Screenshot 2022-01-20 at 3.30.57 PM.png]({{site.baseurl}}/images/Screenshot 2022-01-20 at 3.30.57 PM.png)

Prior to 7.5.1 there is no option to match IPv4/v6 destination addresses for QoS classification on l2transport interface. With introduction to this profile , QoS classification can be done based on destination address even on l2transport interface/sub interface. activation of this profile needs the router to be reloaded. A new match criterion  “match destination-address” is also introduced along with the hw-module profile. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#class-map l2-dest
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#match destination-address ipv4 ?
  A.B.C.D/prefix  IP prefix -network/length-
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#match destination-address ipv4 100.1.1.1/24 ?
  -cr-
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#match destination-address ipv4 100.1.1.1/24
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#match destination-address ipv6 ?
  X:X::X/0-128  IPV6 address with prefix and mask
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#match destination-address ipv6 2001::1/128
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config-cmap)#show configuration
Thu Jan 20 09:54:11.276 UTC
Building configuration...
!! IOS XR Configuration 7.5.1
!
class-map match-any l2-dest
 match destination-address ipv4 100.1.1.1 255.255.255.0
 match destination-address ipv6 2001::1/128
 end-class-map
!
end
</code>
</pre>
</div>

**_hw-module profile qos arp-isis-priority-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile qos ?
  arp-isis-priority-enable        Prioritize ISIS and ARP packets
  conform-aware-policer           Configure Conform Aware Policer mode
  ecn-marking-stats               Enable ECN marking stats mode
  hqos-enable                     Enable Hierarchical QoS on physical and sub-interfaces
  ingress-model                   QoS model for ingress feature
  ipv6                            Configure ipv6 protocol
  l2-match-dest-addr-v4v6         Enable l2qos match on ipv4/ipv6 destination address
  max-classmap-size               max class map size
  physical-hqos-enable            Enable Hierarchical QoS only on physical interfaces, and not on sub-interfaces
  qosg-dscp-mark-enable           Enable both 'set qos-group' and 'set dscp/precedence' actions in the same ingress QoS po
licy
  shared-policer-per-class-stats  Enable shared policer (per class stats) mode
  wred-stats-enable               Enable Wred egress stats
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile qos arp-isis-priority-enable ?
  -cr-
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#hw-module profile qos arp-isis-priority-enable
Thu Jan 20 09:51:01.244 UTC
In order to activate this profile, you must manually reload the chassis/all line cards
</code>
</pre>
</div>
  
![Screenshot 2022-01-20 at 3.30.57 PM.png]({{site.baseurl}}/images/Screenshot 2022-01-20 at 3.30.57 PM.png)

This feature gives an option to assign the highest priority (TC7) to Integrated Intermediate System-to-Intermediate System (IS-IS) and Address Resolution Protocol (ARP) packets in transit. This feature is disabled by default.
The feature provides more flexibility in transit traffic management on a per-hop basis and also fine-tunes the traffic profile management for transit traffic.


### enabling native mode 

**_hw-module profile npu native-mode-enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:IOS(config)#hw-module profile npu ?
  native-mode-enable  Enable NPUs to operate in native mode
RP/0/RP0/CPU0:T-2006(config)#hw-module profile npu

RP/0/RP0/CPU0:ios#config
RP/0/RP0/CPU0:ios(config)#hw-module profile npu native-mode-enable
In order to activate this new npu profile, you must manually reload the chassis
RP/0/RP0/CPU0:ios(config)#commit
RP/0/RP0/CPU0:ios(config)
</code>
</pre>
</div>

J2 based systems can operate in 2 modes: 
Compatible Mode—Used when the chassis contains combination of Cisco NC57 and older generation line cards. This is the default mode.
Native Mode—Used when the chassis contains only Cisco NC57 line cards. 

This hw-module profile is introduced in IOS-XR 7.2.1. For operating a modular chassis with J2 based Line cards in full capacity we need to enable this profile. Jericho and Jericho+ based LC’s present in the chassis should be shut down from admin mode before reload or removed from the chassis. The forwarding behavior will be broken and can be unpredictable if the system will not be able to shut them down properly. To enable the native mode, use the hw-module profile npu native-mode-enable command in the configuration mode. Reload of the router will be needed to switch to native mode. The fixed chassis based on J2 chipset will operate by default in native mode only.

_External documentation_:
-[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/vpn/72x/b-l2vpn-cg-ncs5500-72x/configure-gigabit-ethernet-for-layer-2-VPNs.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/vpn/72x/b-l2vpn-cg-ncs5500-72x/configure-gigabit-ethernet-for-layer-2-VPNs.html)


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

`Note: The encapsulation option is deprecated in 7.4.1 as NCS 5500 will support both SRv6 base and SRv6 uSID modes. Therefore SRv6 related hardware command will follow hw-module profile segment-routing srv6 mode <> and are described below`

**_hw-module profile segment-routing srv6 mode base_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 ?
  encapsulation  Configure encapsulation parameters (DEPRECATED)
  mode           Mode of operation
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode ?
  base           Base SRv6 (Format-1) support only
  micro-segment  Micro-segment support only

RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base ?
  encapsulation  Configure encapsulation parameters
  -cr-
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base
Thu Sep  9 06:23:24.077 UTC
In order to activate/deactivate this srv6 profile, you must manually reload the chassis/all line cards
</code>
</pre>
</div>
 
  ![J-Jp-only-Global.png]({{site.baseurl}}/images/J-Jp-only-Global.png) 
  
 This hw-module mode enables Segment Routing v6 (SRv6) on the node using base SRH format. 
 
 `A reload of the chassis is needed to configure/change any SRv6 related hw-module profile.`
 

 **_hw-module profile segment-routing srv6 mode base encapsulation_**
 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
 RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation ?
  traffic-class  Control traffic-class field on IPv6 header
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation traffic-class ?
  -0x0-0xff-  Traffic-class value (specified as 2 hexadecimal nibbles)
  propagate   Propagate traffic-class from incoming packet/frame
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation traffic-class 0xe0
 </code>
</pre>
</div>
  
 <div class="highlighter-rouge">
<pre class="highlight">
<code>
 RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation ?
  traffic-class  Control traffic-class field on IPv6 header
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation traffic-class ?
  -0x0-0xff-  Traffic-class value (specified as 2 hexadecimal nibbles)
  propagate   Propagate traffic-class from incoming packet/frame
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6 mode base encapsulation traffic-class propagate
 </code>
</pre>
</div>
   
 ![J-Jp-only-Global.png]({{site.baseurl}}/images/J-Jp-only-Global.png) 


The encapsulation option followed by the base SRH format decides the QoS bits for the SRv6 header. The TC value on the IPv6 header can be either set explicitly to any value in the range of 0x00-0xff . This value is global for all the SRv6 services that starts on the box. When configured with propagate, QoS marking of the incoming payload is propagated to the SRv6 TC marking. For L3VPN IP Precedence is copied to the SRv6 header IPv6 precedence. For L2VPN the Layer 2 PCP markings are copied to the SRv6 Precedence. 

`A reload of the chassis is needed to configure/change any SRv6 related hw-module profile.`

**_hw-module profile segment-routing srv6  mode micro-segment format f3216_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment ?
  format  Specify carrier format
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format ?
  f3216  32-bit block and 16-bit IDs
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format f3216 ?
  encapsulation  Configure encapsulation parameters
  -cr-
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format f3216
Wed Aug 25 22:58:12.659 PDT
In order to activate/deactivate this srv6 profile, you must manually reload the chassis/all line cards
   </code>
</pre>
</div>
  
   ![J-Jp-only-Global.png]({{site.baseurl}}/images/J-Jp-only-Global.png) 
  
Introdcued in IOS XR 7.3.1/7.4.1. with this hw-module profile enables SRv6 uSID instead of base SRH format on the node. 
`A reload of the chassis is needed to configure/change any SRv6 related hw-module profile.`


**_hw-module profile segment-routing srv6  mode micro-segment format f3216 encapsulation_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format f3216  encapsulation ?
  traffic-class  Control traffic-class field on IPv6 header
RP/0/RP1/CPU0:5508-1-741C(config)#$ode micro-segment format f3216  encapsulation traffic-class ?
  -0x0-0xff-  Traffic-class value (specified as 2 hexadecimal nibbles)
  propagate   Propagate traffic-class from incoming packet/frame
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format f3216  encapsulation traffic-class 0xe0
Wed Aug 25 23:01:54.477 PDT
In order to activate/deactivate this srv6 profile, you must manually reload the chassis/all line cards
  </code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile segment-routing srv6  mode micro-segment format f3216  encapsulation traffic-class propagate
Wed Aug 25 23:01:59.065 PDT
In order to activate/deactivate this srv6 profile, you must manually reload the chassis/all line cards
  </code>
</pre>
</div>

   ![J-Jp-only-Global.png]({{site.baseurl}}/images/J-Jp-only-Global.png) 
   
   The encapsulation option followed by the micro-segment mode decides the QoS bits for the SRv6 header. The TC value on the IPv6 header can be either set explicitly to any value in the range of 0x00-0xff . This value is global for all the SRv6 services on the box. When configured with propagate, QoS marking of the incoming payload is propagated to the SRv6 TC marking. For L3VPN IP Precedence is copied to the SRv6 header IPv6 precedence. For L2VPN the Layer 2 PCP markings are copied to the SRv6 Precedence.
`A reload of the chassis is needed to configure/change any SRv6 related hw-module profile.`


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

Note: This profiles is not needed for J2 based systems.
{: .notice--info}

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

![Screenshot 2021-04-03 at 12.57.20 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.57.20 PM.png)

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

![Screenshot 2021-04-03 at 12.57.20 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.57.20 PM.png)

Profile increasing the counters available in the egress pipeline from 16K (default) to 24K. These counters are taken from the ingress pipeline, impacting the scale for ACL/QOS/LPTS/etc.   
This command also enables ingress SR counters. QoS stats are disabled while enabling this particular stats profile. 
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

![Screenshot 2021-04-03 at 1.40.07 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 1.40.07 PM.png)


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

![Screenshot 2021-04-03 at 1.41.49 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 1.41.49 PM.png)


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

Note: This profiles is not needed for J2 based systems
{: .notice--info}

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

Note: if you don't configure space for ACL in eTCAM, the hybrid ACL configuration will be rejected at commit. This profile is not needed for J2 based systems.
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

![Screenshot 2021-04-03 at 2.27.25 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 2.27.25 PM.png)


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

![Screenshot 2021-04-03 at 12.54.11 PM.png]({{site.baseurl}}/images/Screenshot 2021-04-03 at 12.54.11 PM.png)


These hardware profiles are necessary to enable the UDK/UDF (User-Defined TCAM Key and Field) used with ingress ACL.  
- They permit the definition of the TCAM key.
- Allow For ttl rewrite based on Tunnel policy if used with set-ttl and match ttl
- Disable ACL sharing and support many small and unique ACL if used with interface-based

Indeed, the default behavior of an ingress access-list is to be re-usable: that means ACE lines of an ingress ACL applied on multiple ports of the same NPU are only counted twice (it's not the case with egress ACLs).  
This hardware profile when used with the keyword "interface-based" will disable the re-usability and move to the unique-ACL mode.

Starting from 6.5.2, we don't support ACL match on packet-lenth (and ranges) by default, it's mandatory to use a specific UDK as described in this blog post:  
[https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/)

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

### mdb

The Modular database (MDB) is applicable to all J2 based (NCS 5700) PIDs. This new hardwre module profile is intoroduced to choose the MDB carving used in J2 Native modes i.e all fixed PIDs based on J2 NPU and all modular NCS 5500 chassis operating in Native mode.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile mdb ?
  l3max     l3max profile for router containing non-TCAM cards
  l3max-se  l3max-se profile for router containing only TCAM cards
  </code>
</pre>
</div>


**_hw-module profile mdb l3max_**
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile mdb l3max ?
  -cr-
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile mdb l3max
Wed Aug 25 23:42:53.961 PDT
In order to activate this new mdb profile, you must manually reload the chassis
  </code>
</pre>
</div>

![J2-Native-Only-Global.png]({{site.baseurl}}/images/J2-Native-Only-Global.png)


introduced in IOS XR 7.4.1, This is applicable to J2 based PIDs (NCS 5700) without any e-TCAM (as well as eTCAM based cards when used in mixed mode). This carves out the NPU modular database to maximize the scale from L3 Heavy deployment. This profile should be applied to all Line Cards in a modular system . l3-max profile is the default MDB profile in a J2 fixed box  or NCS 5500 modular chassis operating in J2 Native mode. 

`To change MDB profiles, the router needs a relaod`

**_hw-module profile mdb l3max-se_**
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile mdb l3max-se ?
  -cr-
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module profile mdb l3max-se
Wed Aug 25 23:43:00.050 PDT
In order to activate this new mdb profile, you must manually reload the chassis
  </code>
</pre>
</div>

![J2-Nat-SE-only-global.png]({{site.baseurl}}/images/J2-Nat-SE-only-global.png)

introduced in IOS XR 7.4.1, This is applicable to J2 based PIDs with external TCAM. This carves out the NPU modular database to maximize the scale from L3 Heavy deployment. For a modular system , all the LCs must be the Scale variant (i.e with eTCAM) for this profile to be applicable. If there is mix of scale and non scale LC, then the system falls back to  l3max profile.


`To change MDB profiles, the router needs a relaod`


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

![]({{site.baseurl}}/images/Screenshot%202021-04-03%20at%202.27.25%20PM.png)

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

## storm-control-combine-policer-bw

**_hw-module storm-control-combine-policer-bw enable_**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module storm-control-combine-policer-bw ?
  enable  Enable storm-control-combine-policer-bw mode
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module storm-control-combine-policer-bw enable ?
  -cr-
RP/0/RP1/CPU0:5508-1-741C(config)#hw-module storm-control-combine-policer-bw enable
Thu Aug 26 22:24:41.022 PDT
In order to activate this storm-control-combine-policer-bw, you must manually reload the chassis
</code>
</pre>
</div>

![J2-Nat-SE-only-global.png]({{site.baseurl}}/images/J2-Nat-SE-only-global.png)

This mode is enabled from IOS XR 7.4.1 and is applicable to J2 based PIDs with external TCAM. For L2 storm control by default the three policer (broadcast, multicast and unknown unicast) are different. This hw-module allows to combine all of them and achieve a combined control late for all BUM traffic. For example, if broadcast storm-control is set to 100 PPS then in default mode, broadcast is limited to 100 PPS and Unknown unicast and Multicast are not subjected to any rate limit. IN the combined mode, the same configuration of a broadcast storm-control policer of 100 pps will rate limit all BUM traffic to 100 PPS. Note that in compbined mode the policer units used must be same for all three policer. If all three policers are configured , then the combined storm control rate is the sum of all three policer.


`Note: To enable/disable the combined policer for storm control using this profile , the router needs to be  reloaded`


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

Bookmark this link and check it regularly for updates.
