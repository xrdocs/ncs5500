---
published: true
date: '2021-09-28 15:34 +0530'
title: Egress IPv6  ACLs on NCS5500
author: Tejas Lad
excerpt: >-
  This document will describe IPv6 ACL Egress ACLs on NCS5500 J/J+ and highlight
  the enhancements when it comes to J2
tags:
  - iosxr
  - NCS5500
  - ncs5500
  - J2
  - Jericho2
  - ACL
  - IPv6
  - Egress ACL
position: top
---
{% include toc icon="table" title="Egress IPv6  ACLs on NCS5500" %} 

## Introduction

In the [previous article](https://xrdocs.io/ncs5500/tutorials/acl-s-on-ncs5500-bvi-interfaces/), we discussed the ACLs on NCS5500 BVI interfaces. We covered the ACL implementation and support for IPv4 and IPv6 ACLs in both ingress and egress directions. We also discussed the enhancements w.r.t IPv6 Egress ACLs. In this article we discuss the overall implementation of IPv6 Egress ACLs across NCS5500 product family based on J/J+ and see how it differs with the platforms based on J2.

## Overview

![Screenshot 2021-09-29 at 2.16.58 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 2.16.58 PM.png)

Let us start with the understanding the PMF block. It is one of the blocks in the ingress and egress pipeline aka Programmable Mapping and Filtering. It is the most programmable and the last programmable block in the pipeline. It has all the history of the packet from other blocks (incoming port, lookup results, etc). We can override here every decision taken along the pipeline. Here we do ACL, QoS, LPTS classification and set actions (counters, policers, Traffic Class).  Egress PMF is capable of doing internal TCAM lookup for egress ACLs

![Screenshot 2021-09-29 at 4.45.23 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 4.45.23 PM.png)

In order to support the match criteria that is needed for egress IPv6 ACL, resources in the Ingress PMF block will be used. Therefore, this requires us to recycle the IPv6 packets so that the ACL can be performed by the Ingress PMF. Sometimes it might happen that certain features will not be processed in a single pass. They will require a second pass in the pipeline for further processing. The recycle interface offers the capability to “re-inject” the packet from the end of the egress pipeline into the ingress pipeline for a “second pass” and may be for a third or fourth if needed. There are 2 types of recycling cases. One is ingress recycle and second is egress recycle. Egress IPv6 ACL is the case of egress recycling.

## Understanding the Two Pass Approach

Egress IPv6 ACL requires two passes of each IPv6 packet that is subject to ACL. The recycling of the IPv6 packets are controlled by entries in the Egress PMF.  Each packet will go through the following stages:

  - 1st pass ingress
  - 1st pass egress (recycle packet)
  - 2nd pass ingress
  - 2nd pass egress

### 1st Pass Ingress

![Screenshot 2021-09-29 at 5.28.14 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-29 at 5.28.14 PM.png)

During the first pass the normal ingress processing will be performed. This includes ingress PMF, QoS, and any other configured features. The forwarding decision, is determined based on the Destination System Port (DSP). Each DSP in the system has a corresponding set of VoQs. Since a FEC (Forwarding Equivalence Class) is pointing to the DSP, packets are sent to destination port on the destination NPU. The header contains DSP as system port. The packets will be put in the DSP’s VoQ and will be scheduled by the DSP’s End-to-End (E2E) scheduler

### 1st Pass Egress

![Screenshot 2021-09-30 at 10.51.26 AM.png]({{site.baseurl}}/images/Screenshot 2021-09-30 at 10.51.26 AM.png)


When an IPv6 ACL is attached to an egress interface, a set of egress PMF entries will created to redirect the IPv6 packet out the recycle port. Based the recycle port configuration, a new program will be selected which will take care of internal processing. At high level it will ensure a fixed packet offset after recycling. It will also build a dummy Ethernet header with Ethertype=IPv6 to prepare for 2nd pass ingress parsing

### 2nd Pass Ingress

![Screenshot 2021-09-30 at 10.59.52 AM.png]({{site.baseurl}}/images/Screenshot 2021-09-30 at 10.59.52 AM.png)


After the IPv6 packet has been recycled, it will be received on the same NPU/Core as the egress DSP. A field group in PMF  will retrieve DSP, Traffic class and drop precedence from the recycled packet's system header. DSP is retrieved so that the recycled packet will go out the same Egress port.  We don't want to go through forwarding again and possibly choose a different egress port. The Traffic Class might have been set by QoS in the 1st pass Ingress processing. We want to preserve this TC. The Drop Precedence might have been set by QoS in the 1st pass ingress processing. We want to preserve this Drop Precedence. Ingress PMF will now perform the ACL. 
If matches on a Deny ACE, the normal processing will occur and the packet will be dropped or  forwarded to the Control Plane for ICMPv6 handling. If matches on a Permit ACE, the configured action will occur plus the additional actions to strip the recycled system header. Forwarding will use the retrieved DSP from the recycled system header. The packets will be put in the DSP’s VoQ and will be scheduled by the DSP’s end to end scheduler **a second time**. 


### 2nd Pass Egress

The original system headers will be used for normal egress processing


## TCAM Entries 

Egress IPv6 ACL has 2 databases:
  - EGRESS_ACL_IPV6 
  - RCY_ACL_L3_IPV6 

Because the packets are recycled, the EGRESS_ACL_IPV6 database has entries that will facilitate the recycling mechanism. The actual match entries are added at RCY_ACL_L3_IPV6. Let us verify the same on the routers. We will use 2 different Line cards as below 


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show platform
Node              Type                       State             Config state
--------------------------------------------------------------------------------               
<mark>0/3/CPU0          NC57-18DD-SE</mark>               IOS XR RUN        NSHUT
0/3/NPU0          Slice                      UP                
0/3/NPU1          Slice                      UP                
<mark>0/4/CPU0          NC55-36X100G-A-SE</mark>          IOS XR RUN        NSHUT
0/4/NPU0          Slice                      UP                
0/4/NPU1          Slice                      UP                
0/4/NPU2          Slice                      UP                
0/4/NPU3          Slice                      UP                                                   
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

We will consider 3 Line cards.

| Line Card         | No of NPU's |
|-------------------|-------------|
| NC57-18DD-SE      | 2xJ2        |
| NC55-36X100G-A-SE | 4xJ+        |

Below ACL is applied on the interface in the egress direction in slot 4.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv6 access-list ipv6_1</mark>
 10 permit tcp 2001:1:2::/64 any eq 1024
 20 permit tcp 2002:1:2::/64 any eq 1024
 30 permit tcp 2003:1:2::/64 any eq 1024
 40 permit tcp 2004:1:2::/64 any eq 1024
 50 deny udp 2001:4:5::/64 any lt 1000
 60 permit ipv6 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-741Cs#show running-config interface hundredGigE 0/4/0/2
Wed Sep 29 22:51:52.131 PDT
<mark>interface HundredGigE0/4/0/2</mark>
 description Local H0/4/0/2  to H0/4/0/10
 <span style="background-color:pink">ipv6 access-group ipv6_1 egress</span>
</code>
</pre>
</div>

As pointed out earlier, we can see 2 databases are created.
<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:5508-2-741Cs#show controllers npu internaltcam location 0/4/CPU0</mark> 
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<mark>0    0</mark>      160b   egress_acl  2026     <span style="background-color:pink">5       31   EGRESS_ACL_IPV6</span>

<mark>0    6\7</mark>    320b   pmf-0       2034     <span style="background-color:pink">14      93   RCY_ACL_L3_IPV6</span>

RP/0/RP0/CPU0:5508-2-741Cs#
</code>
</pre>
</div>

Note: Output is truncated
{: .notice--info}

Let us understand the entries in details:

For database **EGRESS_ACL_IPV6** we have:
- 1 static entry per NPU per core for the recycle-channel
- 3 entries for setting up the different kinds of TPIDs supported (IPv6 (0x86dd), VLAN (0x8100), or MPLS (0x8847))
- We can see 5 entries in the database. (default internal entry + the ones explained above)

For database **RCY_ACL_L3_IPV6** we have:

- The match entries computed from the ACL.
- We can see 14 entries (default entries plus ACEs configured)

Let us increase the number of ACEs and see how the database entries are changed

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv6 access-list ipv6_1</mark>
 10 permit tcp 2001:1:2::/64 any eq 1024
 20 permit tcp 2002:1:2::/64 any eq 1024
 30 permit tcp 2003:1:2::/64 any eq 1024
 40 permit tcp 2004:1:2::/64 any eq 1024
 50 deny udp 2001:4:5::/64 any lt 1000
 60 permit tcp 2005:1:2::/64 any eq 1024
 70 permit tcp 2006:1:2::/64 any eq 1024
 80 permit tcp 2007:1:2::/64 any eq 1024
 90 permit tcp 2008:1:2::/64 any eq 1024
 100 permit ipv6 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-741Cs#show controllers npu internaltcam location 0/4/CPU0 
Wed Sep 29 23:12:18.921 PDT
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
<mark>0    0</mark>      160b   egress_acl  2026     <span style="background-color:pink">5       31   EGRESS_ACL_IPV6</span>

<mark>0    6\7</mark>    320b   pmf-0       2030     <span style="background-color:pink">18      93   RCY_ACL_L3_IPV6</span>

RP/0/RP0/CPU0:5508-2-741Cs#
</code>
</pre>
</div>

We can see for database **EGRESS_ACL_IPV6** the entries are not increasing. The entries are only increasing for the database **RCY_ACL_L3_IPV6**. This is because actual match entries are added only at **RCY_ACL_L3_IPV6**. This is very important to understand during debugging or verifying the outputs. Other thing to remember is, if the same ACL is applied to multiple egress interfaces, match entries and different kinds of TPIDs will increase as per number of interfaces. While the entries for the recycle channel will only increase when the interfaces are in different NPUs.


## J2 Enhancement

NCS5500 based on J2 ASICs do not face the issue of packet recycling when it comes to IPv6 Egress ACLs. They can be processed in single pass like IPv4 Egress ACLs. This is possible because of the presence of more resources in the Egress PMF. Let us verify the same on the Line card based on J2.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>interface FourHundredGigE0/3/0/21</mark>
 description 5508-2 FH0/3/0/21 to 5508-1 FH0/7/0/21
 cdp
 <mark>ipv6 access-group ipv6_1 egress</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-741Cs#show controllers npu internaltcam location 0/3/CPU0   
Wed Sep 29 23:42:35.179 PDT
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================

<mark>1</mark>    6\7    320b   <span style="background-color:pink">EPMF</span>        2013     26      44   <span style="background-color:pink">EGRESS_ACL_IPV6</span>
RP/0/RP0/CPU0:5508-2-741Cs#
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-741Cs#<mark>show controllers npu internaltcam location 0/3/CPU0 | in RCY_ACL_L3_IPV6</mark>

RP/0/RP0/CPU0:5508-2-741Cs#
</code>
</pre>
</div>

From the above outputs, its clearly seen that we no more create a database for recycle port RCY_ACL_L3_IPV6. We can also see that owner of the database is EPMF which is Egress PMF. The Egress PMF is having enough resources to take care of the ACL processing. 

## Summary

Hope this article was helpful in understanding the IPv6 Egress ACL implementation on NCS5500 and the enhancement on the newer generation platforms. In the next article we will understand the various modes of ACLs and see how to use specific profile to increase the ACL scale. Stay tuned !!!
