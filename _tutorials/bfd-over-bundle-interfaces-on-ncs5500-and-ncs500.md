---
published: true
date: '2021-06-18 10:53 +0530'
title: BFD over Bundle Interfaces on NCS5500 and NCS500
author: Tejas Lad
excerpt: This document explains the BFD over Bundle Interfaces on NCS5500 and NCS500
position: top
tags:
  - iosxr
  - NCS5500
  - NCS500
  - BFD
  - BoB
  - Bundle
---
{% include toc icon="table" title="BFD over Bundle Interfaces on NCS55xx and NCS5xx" %} 

## Introduction

We started the NCS5500 and NCS500 BFD technotes with [BFD architecture](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/#:~:text=BFD%20Feature%20Support,-BFD%20is%20supported&text=Static%2C%20OSPF%2C%20BGP%20and%20IS,are%20supported%20in%20IPv4%20BFD.&text=BFD%20over%20BVI%20is%20supported,dampening%20for%20IPv4%20is%20supported.). Then we looked a bit deeper with the [Hardware Programming](https://xrdocs.io/ncs5500/tutorials/understanding-the-bfd-hardware-programming-on-ncs55xx-and-ncs5xx/). We confirmed the behaviour as per [RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880). We also checked the OAM engine programming in the hardware and saw a few commands for quick verification. In this tech-note we will discuss the BFD over Bundles aka BoB. What is the use case and see the programming/behaviour w.r.t [RFC 7130](https://datatracker.ietf.org/doc/html/rfc7130).

## Brief Background


![Screenshot 2021-06-18 at 11.18.09 AM.png]({{site.baseurl}}/images/Screenshot 2021-06-18 at 11.18.09 AM.png)

Just a quick background before we move. Link Bundle or Bundle Ethernet has been in the industry for a long time now. [Link Bundle](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/61x/b-ncs5500-interfaces-configuration-guide-61x/b-ncs5500-interfaces-configuration-guide-61x_chapter_0101.html) is simply a group of ports that are bundled together and act as a single link. The Link Bundling feature allows you to group multiple point-to-point links together into one logical link and provide higher bidirectional bandwidth, redundancy, and load balancing between two routers. A virtual interface is assigned to the bundled link. Cisco IOS XR software supports the IEEE 802.3ad—Standard technology that employs a Link Aggregation Control Protocol (LACP) to ensure that all the member links in a bundle are compatible. The advantages of link bundles are as follows:

  - Multiple links can span several line cards to form a single interface. Thus, the failure of a single link does not cause a loss of connectivity.
  - Bundled interfaces increase bandwidth availability, because traffic is forwarded over all available members of the bundle. Therefore, traffic can flow on the available links if one of the links within a bundle fails. 
  - Bandwidth can be added without interrupting packet flow.
  

## Why do we need BFD over Bundle - BoB ?

As we know, LACP allows a network device to negotiate an automatic bundling of links by sending LACP packets to their directly connected peer. LACP has keep-alive mechanism for link member. The default being 30s and it can be configurable to around 1s. Link Aggregation Control Protocol can
detect failures on a per-physical-member link. However the LACP timers do not fulfill the criteria of today's fast convergence requirements. Therefore using BFD for failure detection would([RFC 7130](https://datatracker.ietf.org/doc/html/rfc7130)): 

  - Provide a faster detection
  - Provide detection in the absence of LACP
  - Would be able to verify the ability for each member link to be able to forward L3 packets.

Running a single BFD session over the aggregation without internal knowledge of the member links would make it impossible for BFD to guarantee detection of the physical member link failures. The goal is to verify link Continuity for every member link.


## BFD over Bundle - BoB on NCS5500 and NCS500 

BFD over Bundle(BoB) implementation on NCS5500 and NCS500 is a standard based fast failure detection of link aggregation (LAG) memberlinks that is interoperable between different platforms. There are 2 modes available: **Cisco and IETF**. NCS5500 and NCS500 only supports the IETF mode. For BFD over Bundle, the BFD client is **bundlemgr**. Hence if BFD session goes down, bundlemgr will bring down the bundle if it violates the minimum link criteria. 

## Configuring BoB

Configuring BoB is very simple. Let us take a simple topology as mentioned above with 3 links connected between 2 routers. We will bundle them into a single virtual interface. Let us configure BFD on the Bundle interfaces as below: 

```
interface Bundle-Ether24
 bfd mode ietf
 bfd address-family ipv4 multiplier 3
 bfd address-family ipv4 destination 192.6.17.17
 bfd address-family ipv4 fast-detect
 bfd address-family ipv4 minimum-interval 300
```

Thats it !!! We are done. We should see BFD neighbor coming up now. Let us verify the same.

## Verifying BoB

```
RP/0/RP0/CPU0:T-2006#show bfd all session 

IPv4:
-----
Interface           Dest Addr           Local det time(int*mult)      State     
                                    Echo             Async   H/W   NPU     
------------------- --------------- ---------------- ---------------- ----------
Hu0/0/1/1           192.6.17.17     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0       
TF0/0/0/24          192.6.17.17     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0       
TF0/0/0/29          192.6.17.17     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0       
BE24                192.6.17.17     n/a              n/a              UP        
                                                             No    n/a 
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd summary 

Node       All PPS usage   MP  PPS usage   Session number    
           %   Used  Max   %   Used  Max   Total MP     Max
---------- --------------- --------------- ------------------
0/0/CPU0   0   0     1000  0   0     700   <mark>3</mark>     0      756
RP/0/RP0/CPU0:T-2006#
</code>
</pre>
</div>

From the above output we can confirm that the total number of sessions are 3. The  Bundle Interface session is a dummy session and it does not count for the overall supported scale on the platform. The session output shows all the member links which can be used to quickly verify the individual members. 

Let us look the detailed output to understand other parameters.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session interface bundle-ether 24 detail 
IPv4:
-----
<mark>I/f: Bundle-Ether24, Location: 0/RP0/CPU0</mark>
Dest: 192.6.17.17
Src: 192.6.17.6
 <mark>State: UP for 0d:1h:21m:53s, number of times UP: 1</mark>
 <mark>Session type: PR/V4/SH/BI/IB</mark>
 Session owner information:                          Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib</mark>    300 ms     3          300 ms     3         
Session association information:
 <mark> Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  TF0/0/0/24           192.6.17.17                             
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  TF0/0/0/29           192.6.17.17                             
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER   
  Hu0/0/1/1            192.6.17.17                             
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_MEMBER </mark>
</code>
</pre>
</div>

We can see the Bundle Interface 24 as the BFD interface with session owners as the individual member links. The session type is PR/V4/SH/BI/IB. 

| Flags | Session Type                                                                                                            |
|-------|-------------------------------------------------------------------------------------------------------------------------|
| PR    | Pre-Routed Session mostly single path sessions applicable for Physical or Sub-interfaces and BFD over Bundle interfaces |
| V4    | IPv4 Session                                                                                                            |
| SH    | Single Hop Session                                                                                                      |
| BI    | Bundle Interface                                                                                                        |
| IB    | IETF BoB                                                                                                                |

But the above output does not give us the full details on discriminators, negotiated timer values so-on and so-forth. For looking into those values individually, we need to check the below command.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session detail interface <mark>twentyFiveGigE 0/0/0/24</mark>    
IPv4:
-----
<mark>I/f: TwentyFiveGigE0/0/0/24, Location: 0/0/CPU0</mark>
Dest: 192.6.17.17
Src: 192.6.17.6
 State: UP for 0d:1h:39m:22s, number of times UP: 1
 <mark>Session type: PR/V4/SH/BM/IB</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147491864, your discr: 2147487752, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147487752, your discr: 2147491864, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Timer Values:</mark>
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=4, min=5 ms, max=14 s, avg=7164 ms
       Last packet transmitted 5949 s ago
   Rx: Number of intervals=13, min=2 ms, max=1700 ms, avg=1195 ms
       Last packet received 5962 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>Session owner information:</mark>
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bundlemgr_distrib</mark>    300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  <mark>BE24                 192.6.17.17                             
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_INTERFACE</mark>
<mark>H/W Offload Info:</mark>
 <mark>H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0</mark>
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 49/35          
Platform Info:
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x1008     Echo Session ID    : 0x0
Async Tx Key        : 0x80001008  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
</code>
</pre>
</div>

| Flags | Session Type                                                                                                            |
|-------|-------------------------------------------------------------------------------------------------------------------------|
| PR    | Pre-Routed Session mostly single path sessions applicable for Physical or Sub-interfaces and BFD over Bundle interfaces |
| V4    | IPv4 Session                                                                                                            |
| SH    | Single Hop Session                                                                                                      |
| BM    | Bundle Member                                                                                                        |
| IB    | IETF BoB                                                                                                                |

Similary we can check the parameters of the other bundle members as well. (show bfd all session detail interface twentyFiveGigE 0/0/0/29, show bfd all session detail interface hun 0/0/1/1)

## BFD over Bundle with IPv4 Unnumbered Interfaces

With [IOSXR-7.3.1](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/73x/b-routing-cg-ncs5500-73x/implementing-bfd.html#Cisco_Concept.dita_fa9251b5-0c63-4382-ba40-d591ad8e1925), NCS5500 and NCS500 supports BFD over Bundle with IPv4 Unnumbered Interfaces . This feature enables BFD to run on IP unnumbered interfaces, which take the IP address from the loopback address. The same loopback address is used on multiple interfaces. This helps to save IP addresses space or range. BFD creates a session on the unnumbered interface for which the BFD clients provide the source and destination IP address along with the interface index. BFD establishes the session on the Layer 3 unnumbered link to which the interface index corresponds. The source address is derived from the Loopback interface at the source. The destination node also uses IP unnumbered interface with loopback address and that is used as destination IP address. BFD sends control packets to the unnumbered interfaces. These control packets are the regular IP BFD packets. Address Resolution Protocol (ARP) resolves the destination loopback IP address to the destination node’s router MAC address. Let us verify the same

### Configuring the session over IPv4 unnumbered interfaces

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show running-config interface bundle-ether 24

interface Bundle-Ether24
 bfd mode ietf
 bfd address-family ipv4 multiplier 3
 bfd address-family ipv4 destination 172.16.4.41
 bfd address-family ipv4 fast-detect
 bfd address-family ipv4 minimum-interval 300
 ipv4 point-to-point
 <mark>ipv4 unnumbered Loopback0</mark>
 bundle minimum-active links 1
!

RP/0/RP0/CPU0:T-2006#
</code>
</pre>
</div>

### Verifying the BFD session

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session    

IPv4:
-----
Interface           Dest Addr           Local det time(int*mult)      State     
                                    Echo             Async   H/W   NPU     
------------------- --------------- ---------------- ---------------- ----------
TF0/0/0/24          172.16.4.41     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0       
TF0/0/0/29          172.16.4.41     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0       
BE24                172.16.4.41     n/a              n/a              UP        
                                                             No    n/a 
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session interface <mark>twentyFiveGigE 0/0/0/24</mark> detail 

IPv4:
-----
<mark>I/f: TwentyFiveGigE0/0/0/24, Location: 0/0/CPU0</mark>
<mark>Dest: 172.16.4.41
Src: 172.16.4.6</mark>
 <mark>State: UP for 0d:0h:5m:36s, number of times UP: 1
 Session type: PR/V4/SH/BM/IB</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147491940, your discr: 2147487763, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147487763, your discr: 2147491940, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=6, min=110 ms, max=62 s, avg=19 s
       Last packet transmitted 241 s ago
   Rx: Number of intervals=19, min=2 ms, max=65 s, avg=6186 ms
       Last packet received 240 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
 <mark> bundlemgr_distrib</mark>    300 ms     3          300 ms     3         
Session association information:
  Interface            Dest Addr / Type                   
  -------------------- -----------------------------------
  BE24                 172.16.4.41                             
                       BFD_SESSION_SUBTYPE_RTR_BUNDLE_INTERFACE

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 130/67 
          
Platform Info:
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x1013     Echo Session ID    : 0x0
Async Tx Key        : 0x80001013  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
</code>
</pre>
</div>

From the above output we can see that the source and destination value taken now, is of the loopback interface. We have not applied any IPv4 address on the Bundle Interface.


## Platform Support

| Platform | Support |
|----------|---------|
| NCS5500  | Yes     |
| NCS540   | Yes     |
| NCS560   | Yes     |

Note: BFD over Bundle is supported with mixed speed interfaces.
{: .notice--info}

## Summary

Hope this article helped to understand the BoB feature on the NCS5500 and NCS500 and its use case. We have also seen the basic commands for quick verification. We also covered the BoB with IP Unnumbered interface. In the next article, we will cover the multipath sessions: BLB and BVI and understand their use cases.

## References 

[CCO Guide ](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/73x/b-routing-cg-ncs5500-73x/implementing-bfd.html#Cisco_Concept.dita_fa9251b5-0c63-4382-ba40-d591ad8e1925)
