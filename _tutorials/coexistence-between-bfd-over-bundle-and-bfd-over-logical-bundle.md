---
published: true
date: '2021-08-06 14:14 +0530'
title: Coexistence Between BFD over Bundle and BFD over Logical Bundle
position: top
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
  
To overcome these limitations, it is possible to run BoB and BLB in parallel on the same bundle interface. This provides the faster bundle convergence from BoB and the true L3 check from BLB ([Reference](https://community.cisco.com/t5/service-providers-documents/bfd-on-crs/ta-p/3154501))

## BoB and BLB Coexistence Feature Support

  - The feature is supported from IOS-XR 741.
  - It is supported on all the platforms including **NCS 540**, **NCS 560** and **NCS 5500** (including systems based on J2 Native and Compatible modes).
  - Support for two modes: Inherit and Logical
  - Supported for BFD v4 and v6 sessions

## Inherit and Logical Modes of Operation([Reference](https://community.cisco.com/t5/service-providers-documents/bfd-on-crs/ta-p/3154501))

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

As we saw in the earlier section, when the **Inherit** mode is configured, the BLB session inherits from BoB session, as the BLB session is created as a virtual session, but never gets downloaded to the LC and so, no real packets are sent out.  The “logical” mode creates a BFD session with real packets and BLB requires multi-path (MP) configurations irrespective of the inherited mode. Inherited sessions when BOB is not enabled will be held in DOWN State. Let us verify the same.

We have BoB configured on the interface and can see the session is UP.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show running-config interface bundle-ether 30
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
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether30</mark>, Location: 0/RP1/CPU0
Dest: 30.1.1.2
Src: 30.1.1.1
 <mark>State: UP for 1d:18h:48m:46s, number of times UP: 2
 Session type: PR/V4/SH/BI/IB</mark>
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  bundlemgr_distrib    300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  <mark>FH0/7/0/23           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  FH0/7/0/21           30.1.1.2                                
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  BE30.1               31.1.1.2                                
                       BFD_SESSION_SUBTYPE_STATE_INHERIT</mark>
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
 <mark>State: UP for 0d:0h:7m:58s, number of times UP: 1
 Session type: PR/V4/SH/IH</mark>
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

Let us remove the BoB configs 

```
interface Bundle-Ether30
 description connected to NCS5508-2
 ipv4 address 30.1.1.1 255.255.255.252
!
```

We can see there is no BoB session now.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30 detail 


IPv4:
-----
</code>
</pre>
</div>

The BLB session has gone down as there is no way to inherit the BFD packets.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30.1 detail  


IPv4:
-----
<mark>I/f: Bundle-Ether30.1</mark>, Location: 0/RP1/CPU0
Dest: 31.1.1.2
Src: 31.1.1.1
 <mark>State: DOWN</mark> for 0d:0h:4m:46s, number of times UP: 1
 Session type: PR/V4/SH/IH
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  isis-1               300 ms     3          300 ms     3         
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:Aug  9 21:17:09.336 PDT: bfd[1269]: %L2-BFD-6-SESSION_INHERIT_OFF : BFD session to neighbor 31.1.1.2 on interface Bundle-Ether30.1 is not inheriting state from session over Bundle-Ether30 
RP/0/RP1/CPU0:Aug  9 21:17:09.336 PDT: bfd[1269]: %L2-BFD-6-SESSION_STATE_DOWN : BFD session to neighbor 31.1.1.2 on interface Bundle-Ether30.1 has gone down. Reason: Admin down 
RP/0/RP1/CPU0:Aug  9 21:17:09.336 PDT: bfd[1269]: %L2-BFD-6-SESSION_BUNDLE_ALL_MEMBER_OFF : BFD session to neighbor 30.1.1.2 on interface Bundle-Ether30 is not running in all member mode 
RP/0/RP1/CPU0:Aug  9 21:17:09.336 PDT: bfd[1269]: %L2-BFD-6-SESSION_REMOVED : BFD session to neighbor 30.1.1.2 on interface Bundle-Ether30 has been removed 
LC/0/7/CPU0:Aug  9 21:17:09.337 PDT: bfd_agent[342]: %L2-BFD-6-SESSION_REMOVED : BFD session to neighbor 30.1.1.2 on interface FourHundredGigE0/7/0/23 has been removed 
LC/0/7/CPU0:Aug  9 21:17:09.337 PDT: bfd_agent[342]: %L2-BFD-6-SESSION_REMOVED : BFD session to neighbor 30.1.1.2 on interface FourHundredGigE0/7/0/21 has been removed 
RP/0/RP1/CPU0:Aug  9 21:17:10.439 PDT: config[66222]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'root'. Use 'show configuration commit changes 1000000488' to view the changes. 
RP/0/RP1/CPU0:Aug  9 21:17:52.509 PDT: isis[1009]: %ROUTING-ISIS-5-ADJCHANGE : Adjacency to 5508-2-74142I-C (Bundle-Ether30.1) (L2) Down, Neighbor forgot us 
RP/0/RP1/CPU0:Aug  9 21:18:35.402 PDT: config[66222]: %MGBL-SYS-5-CONFIG_I : Configured from console by root on vty1 (192.168.100.151) 
RP/0/RP1/CPU0:5508-1-74142I-C#          
</code>
</pre>
</div>

Changing the coexistence mode back to logical. We can see that the BLB session has come up, because during this mode real packets are exchanged and we do not inherit it from the BoB.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP1/CPU0:5508-1-74142I-C#show running-config bfd
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
RP/0/RP1/CPU0:5508-1-74142I-C#show bfd all session interface bundle-ether 30.1 detail 

IPv4:
-----
<mark>I/f: Bundle-Ether30.1</mark>, Location: 0/7/CPU0
Dest: 31.1.1.2
Src: 31.1.1.1
 <mark>State: UP for 0d:0h:0m:29s, number of times UP: 1
 Session type: SW/V4/SH/BL</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 7340052, your discr: 15728652, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 15728652, your discr: 7340052, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Label:
 Internal label: 24025/0x5dd9
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=6 ms, max=18 s, avg=6805 ms
       Last packet transmitted 29 s ago
   Rx: Number of intervals=5, min=3 ms, max=1680 ms, avg=520 ms
       Last packet received 29 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>MP download state: BFD_MP_DOWNLOAD_ACK</mark>
State change time: Aug  9 21:25:57.190
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>isis-1</mark>               300 ms     3          300 ms     3         

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/7/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 41/14 

<mark>Platform Info:</mark>
NPU ID: 1 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0xc        Echo Session ID    : 0x0
Async Tx Key        : 0xf0000c  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
LC/0/7/CPU0:Aug  9 21:25:57.189 PDT: bfd_agent[342]: %L2-BFD-6-SESSION_DAMPENING_ON : Session to neighbor 31.1.1.2 on interface Bundle-Ether30.1 entered Dampened state (initial: 2000 ms,secondary: 5000 ms,maximum: 120000 ms). 
RP/0/RP1/CPU0:Aug  9 21:25:59.137 PDT: config[66604]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'root'. Use 'show configuration commit changes 1000000489' to view the changes. 
LC/0/7/CPU0:Aug  9 21:26:15.307 PDT: bfd_agent[342]: %L2-BFD-6-SESSION_DAMPENING_OFF : Session to neighbor 31.1.1.2 on interface Bundle-Ether30.1 moved out of Dampened state. 
LC/0/7/CPU0:Aug  9 21:26:17.601 PDT: bfd_agent[342]: %L2-BFD-6-SESSION_STATE_UP : BFD session to neighbor 31.1.1.2 on interface Bundle-Ether30.1 is up 
RP/0/RP1/CPU0:Aug  9 21:26:18.799 PDT: isis[1009]: %ROUTING-ISIS-5-ADJCHANGE : Adjacency to 5508-2-74142I-C (Bundle-Ether30.1) (L2) Up, Restarted 
</code>
</pre>
</div>


## Which mode to choose: Inherit vs Logical ?

While choosing between the coexistence mode, we need to understand how each mode works and what do we want to achieve. As we saw in the earlier sections and examples, in Inherit mode, BLB sessions will simply inherit the BoB session state and doesn’t have its own state machine. If BoB is down then BLB also will go down. BLB clients (routing protocols) are tied to BoB session state. So if BoB session goes down, along with bundle-mgr, routing protocols too get notified. In the Logical mode BLB and BoB sessions are independent sessions and notify their respective clients based on their timeout configured. If link failure detection is the only criteria for customer then BoB should be enough in most cases. But convergence may get affected due to the point of indirect notification to the routing protocols. If forwarding detection is also required for the bundle main/sub-interfaces, then BLB should be used along with BoB. This will help in achieving faster convergence.

## Memory and Scale Impact

There is no memory impact expected while configuring the BoB and BLB coexistence. When using inherit mode the scale will only be considered for the BoB sessions. With logical mode it would be a summation of BoB and BLB. The overall scale calculation will not change and remain as per the existing scale supported. For details on how the scale is calculated please [refer](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/).


## Summary

Hope you find this article useful. We covered a quick background BoB and BLB and what are their limitations are when using either of them. We saw how the BoB and BLB coexistence can help BFD converge faster and give better results. We saw the configuration examples along with igp as ISIS. This feature is supported with OSPF, BGP as well as static. We discussed on what basis we can select the coexistence mode. As a best practice, for faster convergence we should use BoB-BLB coexistence wherever possible.
