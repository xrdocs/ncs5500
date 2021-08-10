---
published: true
date: '2021-08-06 14:14 +0530'
title: Coexistence Between BFD over Bundle and BFD over Logical Bundle
position: hidden
author: Tejas Lad
excerpt: >-
  This document will cover the Coexistence Between BFD over Bundle and BFD over
  Logical Bundle and why it is needed
tags:
  - iosxr
  - BFD
  - BLB
  - BOB
  - NCS 5500
  - NCS 500
  - BLB BoB Coexistence
---
{% include toc icon="table" title="Coexistence Between BFD over Bundle and BFD over Logical Bundle" %} 

## Introduction

In the previous articles, we introduced the concepts of [BFD over Bundle](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/) and [BFD over Logical Bundle](https://xrdocs.io/ncs5500/tutorials/multipath-and-multihop-bfd-sessions-on-ncs5500-and-ncs500/). We saw the configurations and the use cases. In this article we will discuss their limitations, when they are used one-at-a-time and see the why we need their coexistence.

## Quick recap

![Screenshot 2021-08-06 at 2.45.40 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-06 at 2.45.40 PM.png)


BFD over Bundle(BoB) implementation is a standard based fast failure detection of link aggregation (LAG) member links that is interoperable between different platforms. BFD over Bundle implements BFD per member link. Whereas the Bidirectional Forwarding Detection (BFD) over Logical Bundle feature implements and deploys BFD over bundle interfaces. This is the fundamental difference between BLB and BoB. In BLB the bundle interface is a single interface, whereas in the BoB we implement BFD per member link. BLB is a multipath (MP) single-hop session.If BLB is running on a bundle there is only one BFD session running. This implies that only one bundle-member is being monitored by BFD, at any given time. Whereas in case of BoB, we have BFD sessions equal to the number of member links.

## Limitations 

  - BoB does not provide true L3 check and is not supported on subinterfaces.
  - With BLB, a failure of bundle members, which BFD is not running on is not detected.
  - And a failure of a bundle member, which BFD is running on will cause BFD to declare a session     failure on the bundle, even if there are sufficient numbers of other bundle members available     and functional.
  
To overcome these limitations, it is possible to run BoB and BLB in parallel on the same bundle interface. This provides the faster bundle convergence from BoB and the true L3 check from BLB ([Reference](https://kxiwq67737.lithium.com/t5/service-providers-documents/bfd-on-crs/ta-p/3154501))

## BoB and BLB Coexistence Feature Support

  - The feature is supported from IOS-XR 741.
  - It is supported on all the platforms including **NCS 540**, **NCS 560** and **NCS 5500** (including systems based on J2).
  - Support for two modes: Inherit and Logical
  - Supported for BFD v4 and v6 sessions

## Inherit and Logical Modes of Operation([Reference](https://kxiwq67737.lithium.com/t5/service-providers-documents/bfd-on-crs/ta-p/3154501))

BoB and BLB coexistence is supported with the following 2 modes: 

**Inherit**: When the “inherit” coexistence mode is configured then a BLB will always create a virtual session and never a BFD session with real packets. This means BLB will not send packets, but will refer the packets that are sent by BoB. 

**Logical**: When the option "logical" is used BLB will always create a real session even when BoB is on. There is one exception if the main bundle interface has an IPv4 address. In this case the session is inherited when BoB is on.

We will see this is details when we check it on our routers.

## Router Demo

After all the theory behind the need the BoB and BLB coexistence, let us see it in action. We will use 2 back to back connected routers with IOS-XR 741 and verify different scenarios with inherit and logical mode. Below is our lab setup.

![Screenshot 2021-08-06 at 3.20.44 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-06 at 3.20.44 PM.png)

### Inherit Mode 

Let us first configure the linecards to allow hosting of MP BFD sessions. If no linecards are included, linecards groups are not formed, and consequently no BFD MP sessions are created. And second configuration is related to the BoB-BLB coexistence. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>bfd
 multipath include location 0/7/CPU0
 bundle coexistence bob-blb inherit</mark>
!
</code>
</pre>
</div>

**BoB and BLB configs** 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface Bundle-Ether30
 description connected to NCS5508-2
 bfd mode ietf
 bfd address-family ipv4 multiplier 3
 bfd address-family ipv4 destination 30.1.1.2
 bfd address-family ipv4 fast-detect
 bfd address-family ipv4 minimum-interval 300
 ipv4 address 30.1.1.1 255.255.255.252
!
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router isis 1
 is-type level-2-only
 net 49.0000.0000.0191.00
 log adjacency changes
 log pdu drops
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
!
 interface Bundle-Ether30
  bfd minimum-interval 300
  bfd multiplier 3
  bfd fast-detect ipv4
  point-to-point
  address-family ipv4 unicast
  !
</code>
</pre>
</div>

**Verifying the BFD sessions**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session
IPv4:
-----
Interface           Dest Addr           Local det time(int*mult)      State     
                                    Echo             Async   H/W   NPU     
------------------- --------------- ---------------- ---------------- ----------
<mark>FH0/7/0/23          30.1.1.2        0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/7/CPU0       
FH0/7/0/21          30.1.1.2        0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/7/CPU0                
BE30                30.1.1.2        n/a              n/a              UP        
                                                             No    n/a </mark> 
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30</mark>, Location: 0/RP1/CPU0
Dest: 30.1.1.2
Src: 30.1.1.1
 State: UP for 0d:0h:0m:51s, number of times UP: 2
 <mark>Session type: PR/V4/SH/BI/IB</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib    300 ms     3          300 ms     3 </mark>        
  <mark>isis-1               300 ms     3          300 ms     3 </mark>        
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  <mark>FH0/7/0/23           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  FH0/7/0/21           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER</mark> 
</code>
</pre>
</div>

| Flags | Session Type                                                                                                            |
|-------|-------------------------------------------------------------------------------------------------------------------------|
| PR    | Pre-Routed Session mostly single path sessions applicable for Physical or Sub-interfaces and BFD over Bundle interfaces |
| V4    | IPv4 Session                                                                                                            |
| SH    | Single Hop Session                                                                                                      |
| BI    | Bundle Interface                                                                                                        |
| IB    | IETF BoB                                                                                                                |

Now let us add a sub-interface of the same bundle for the ISIS adjacency. We will keep the mode as Inherit.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router isis 1
 is-type level-2-only
 net 49.0000.0000.0191.00
 address-family ipv4 unicast
  metric-style wide
 !
 <mark>interface Bundle-Ether30.1</mark>
  bfd minimum-interval 300
  bfd multiplier 3
  bfd fast-detect ipv4
  point-to-point
  address-family ipv4 unicast
  !
 !
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30</mark>, Location: 0/RP1/CPU0
Dest: 30.1.1.2
Src: 30.1.1.1
 State: UP for 0d:0h:12m:47s, number of times UP: 2
 <mark>Session type: PR/V4/SH/BI/IB</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib</mark>    300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  FH0/7/0/23           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  FH0/7/0/21           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  <mark>BE30.1               31.1.1.2                                
                       BFD_SESSION_SUBTYPE_STATE_INHERIT </mark>      
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30.1 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30.1</mark>, Location: 0/RP1/CPU0
Dest: 31.1.1.2
Src: 31.1.1.1
 State: UP for 0d:0h:1m:59s, number of times UP: 1
 <mark>Session type: PR/V4/SH/IH</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  isis-1               300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  <mark>BE30                 30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_INTERFACE</mark>
</code>
</pre>
</div>

| Flags | Session Type                                                                                                            |
|-------|-------------------------------------------------------------------------------------------------------------------------|
| PR    | Pre-Routed Session mostly single path sessions applicable for Physical or Sub-interfaces and BFD over Bundle interfaces |
| V4    | IPv4 Session                                                                                                            |
| SH    | Single Hop Session                                                                                                      |
| IH    | State Inherit                                                                                                           |


As we discussed above, we can see that in the Inherit mode, BLB will always create a virtual session and never a BFD session with real packets. 


### Logical Mode

Let us verify the behaviour in the logical mode. We will change the configuration of the coexistence mode as logical in place of inherit.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
bfd
 multipath include location 0/7/CPU0
 <mark>bundle coexistence bob-blb logical</mark>
!
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show bfd all session interface bundle-ether 30 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30</mark>, Location: 0/RP0/CPU0
Dest: 30.1.1.1
Src: 30.1.1.2
 State: UP for 0d:0h:50m:32s, number of times UP: 3
 <mark>Session type: PR/V4/SH/BI/IB</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib</mark>    300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  <mark>FH0/3/0/23           30.1.1.1                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  FH0/3/0/21           30.1.1.1                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER</mark> 
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show bfd all session interface bundle-ether 30.1 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30.1</mark>, Location: 0/3/CPU0
Dest: 31.1.1.1
Src: 31.1.1.2
 State: UP for 0d:0h:5m:28s, number of times UP: 1
 <mark>Session type: SW/V4/SH/BL</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 15728648, your discr: 7340048, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 7340048, your discr: 15728648, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Label:
 Internal label: 24036/0x5de4
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=7 ms, max=5103 ms, avg=1802 ms
       Last packet transmitted 327 s ago
   Rx: Number of intervals=8, min=5 ms, max=1700 ms, avg=712 ms
       Last packet received 327 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>MP download state: BFD_MP_DOWNLOAD_ACK</mark>
State change time: Aug  8 03:11:10.233
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>isis-1</mark>               300 ms     3          300 ms     3         

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/3/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 54/19 

Platform Info:
NPU ID: 1 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x10       Echo Session ID    : 0x0
Async Tx Key        : 0x700010  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0

</code>
</pre>
</div>

From the above output we can see that, when the option "logical" is used BLB will always create a real session. We can see the BFD packets are exchanged between the two neighbors and BLB Flag is set. 

When we configure the BoB-BLB coexistence in Logical mode, there is one exception. If the main bundle interface has an IPv4 address, the session is inherited when BoB is on. Let us configure a BLB session via a bundle sub-interface.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30</mark>, Location: 0/RP1/CPU0
Dest: 30.1.1.2
Src: 30.1.1.1
 State: UP for 0d:0h:41m:54s, number of times UP: 2
 <mark>Session type: PR/V4/SH/BI/IB</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib    300 ms     3          300 ms     3         
  isis-1               300 ms     3          300 ms     3</mark>         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  FH0/7/0/23           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  FH0/7/0/21           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER  
</code>
</pre>
</div>


## BLB dependency on BoB in Inherit Mode



## Which mode to choose: Inherit vs Logical ?

When the “inherited” mode is configured, the BLB session inherits from BoB session, as the BLB session is created as a virtual session, but never gets downloaded to the LC and so, no real packets are sent out.  The “logical” mode creates a BFD session with real packets and BLB requires multi-path (MP) configurations irrespective of the inherited mode. Inherited sessions when BOB is not enabled will be held in DOWN State. 

## Memory and Scale Impact

## Summary

We covered a quick background BoB and BLB and what are their limitations are when using either of them. We also saw how the BoB and BLB coexistence can help BFD converge faster and give better results. We saw the configuration examples along with igp as ISIS. This is supported with OSPF, BGP as well as static. Hope this article was helpful.
