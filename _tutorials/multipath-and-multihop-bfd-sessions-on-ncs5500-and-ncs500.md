---
published: true
date: '2021-06-22 12:12 +0530'
title: Multipath and MultiHop BFD sessions on NCS5500 and NCS500
author: Tejas Lad
excerpt: >-
  This document covers the implementation of BLB and BFD over BVI on NCS5500 and
  NCS500
tags:
  - iosxr
  - NCS5500
  - NCS500
  - NCS 5500
  - BFD
  - BLB
  - BVI
position: top
---
{% include toc icon="table" title="Multipath BFD sessions on NCS5500 and NCS500" %} 

## Introduction

In our previous article we discussed [BFD over Bundle Interface or BoB](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/). In this article, we discuss one more concept for bundle: BFD over Logical Bundle or BLB. We will understand the difference between BLB and BoB and see its use cases. We will also see what are Multipath and Multihop Sessions with BFD over BVI interfaces.

## BFD over Logical Bundle

With IOS-XR 7.1.1, we support BLB on NCS5500 and NCS500 platforms. The Bidirectional Forwarding Detection (BFD) over Logical Bundle feature implements and deploys BFD over bundle interfaces based on RFC 5880. This is the fundamental difference between BLB and BoB. In the former the bundle interface is a single interface, whereas in the later we implement BFD per member link. BLB is a multipath (MP) single-hop session. BLB requires limited knowledge of the bundle interfaces on which the sessions run, this is because BFD treats the bundle as one big pipe. To function, [BLB](https://www.cisco.com/c/en/us/td/docs/routers/asr9000/software/asr9k-r6-2/routing/configuration/guide/b-routing-cg-asr9000-62x/b-routing-cg-asr9000-62x_chapter_0100.html#concept_45D0EC68B0A241B39338087FEB10055F) requires only information about IP addresses, interface types, and caps on bundle interfaces. Information such as list of bundle members, member states, and configured minimum or maximum bundle links are not required. In case of BLB, BFD client is not Bundle link but protocols running over bundle link. BLB is supported on IPv4 and IPv6 addresses and IPv6 link-local address.  

| BFD Session Type              | BFD Client                        |
|-------------------------------|-----------------------------------|
| BFD over Bundle - BoB         | bundlemgr                         |
| BFD over Logical Bundle - BLB | protocols running over the bundle |


### Configuring BLB 

![Screenshot 2021-06-22 at 2.29.37 PM.png]({{site.baseurl}}/images/Screenshot 2021-06-22 at 2.29.37 PM.png)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router isis 1
 is-type level-2-only
 net 49.0000.0000.0000.0006.00
 address-family ipv4 unicast
  metric-style wide
 !
 <mark>interface Bundle-Ether24</mark>
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

The BFD instance will run locally on the line card CPU. Therefore, we must manually designate the line card CPU on which the BFD session will be run. If the bundle members are spread across different locations, we need to specify all of them. 

```
bfd
 multipath include location 0/0/CPU0
!
```

### Verifying BLB 

```
RP/0/RP0/CPU0:T-2006#show bfd all session 

IPv4:
-----
Interface           Dest Addr           Local det time(int*mult)      State     
                                    Echo             Async   H/W   NPU     
------------------- --------------- ---------------- ---------------- ----------
BE24                192.6.17.17     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0 
```

From the above output we can see, it shows a single session over the bundle interface. If we compare it with [BoB](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/), we can see there were individual member links in the session. Let us analyse the detailed output.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session interface bundle-ether 24 detail 

IPv4:
-----
<mark>I/f: Bundle-Ether24, Location: 0/0/CPU0</mark>
Dest: 192.6.17.17
Src: 192.6.17.6
 <mark>State: UP for 0d:0h:17m:25s, number of times UP: 1</mark>
 <mark>Session type: SW/V4/SH/BL</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 4, your discr: 1, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 1, your discr: 4, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Label:
 Internal label: 24017/0x5dd1
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=14 ms, max=299 ms, avg=115 ms
       Last packet transmitted 1045 s ago
   Rx: Number of intervals=4, min=3 ms, max=299 ms, avg=153 ms
       Last packet received 1045 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>MP download state: BFD_MP_DOWNLOAD_ACK</mark>
State change time: Jun 22 09:30:22.849
<mark>Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  isis-1               300 ms     3          300 ms     3  </mark>       

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 19/4 

Platform Info:
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x1        Echo Session ID    : 0x0
Async Tx Key        : 0x1  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0

</code>
</pre>
</div>

| Flags | Session Type                                                 |
|-------|--------------------------------------------------------------|
| SW    | Switched Session which includes BFD over Logical Bundle- BLB |
| V4    | IPv4 Session                                                 |
| SH    | Single Hop Session                                           |
| BL    | BFD Type is BLB                                              |

From the detailed output we can see that its a switched session with session type as BLB. The BFD client is IS-IS. We can also see **MP download state: BFD_MP_DOWNLOAD_ACK** which shows its a Multi-path session. Like IS-IS we can also use the OSPF and Static route as clients. We can also use vlan sub-interfaces along with physical main interface in the routing process. We can see BLB is a single hop multipath session. Source/Destination MAC address for BFD control packet is MAC of Bundle-Ether interface. BLB treats the bundle as one single interface and BFD packet may take any physical link to reach other end device.


## BFD over BVI Interface

From IOS-XR 7.1.1, BFD is supported over BVI Interface. Let us start with a quick background of [BVI Interface](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/71x/b-routing-cg-ncs5500-71x/b-routing-cg-ncs5500-71x_chapter_0111.html#concept_hdq_sfm_gkb). In order for a VLAN to span a router, the router must be capable of forwarding frames from one interface to another, while maintaining the VLAN header. If the router is configured for routing a Layer 3 (network layer) protocol, it will terminate the VLAN and MAC layers at the interface on which a frame arrives. The MAC layer header can be maintained if the router bridges the network layer protocol. However, even regular bridging terminates the VLAN header. Using this feature, a router can be configured for routing and bridging the same network layer protocol, on the same interface. This allows the VLAN header to be maintained on a frame while it transits a router from one interface to another. The BVI is a virtual interface within the router that acts like a normal routed interface that does not support bridging, but represents the comparable bridge group to routed interfaces within the router. The interface number of the BVI is the number of the bridge group that the virtual interface represents. This number is the link between the BVI and the bridge group. Because the BVI represents a bridge group as a routed interface, it must be configured only with Layer 3 (L3) characteristics, such as network layer addresses. Similarly, the interfaces configured for bridging a protocol must not be configured with any L3 characteristics. 

With IOS-XR 7.1.1 and beyond we can configure IPv4/IPv6 BFD over BVI  interface with OSFP/ISIS/Static/BGP clients. BFD over BVI is a single hop (SH) multipath (MP)session. The details are very similar to existing multipath-based. BFD Packet Reception on BVI interfaces is similar to MP session. In the first pass, routing occurs as destination mac in received packet matching with bvi interface mac and packet is considered as a for us packets if the destination IP address is matching with BVI interface IP address. In the second pass BFD processes a packet and sends it to OAMP/ARM block. This is same way how packet will be processed as packet received on physical interface. BFD over BVI will be supported on IPv4 address, IPv6 global address and IPv6 link-local addresses. For details on BFD architecture and packet flow please [visit](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/).

### Configuring BFD over BVI 

```
interface Bundle-Ether24
 bundle minimum-active links 1
 l2transport
 !
 
l2vpn
 bridge group 24
  bridge-domain BE24
   interface Bundle-Ether24
   !
   routed interface BVI24
   !

router isis 1
 is-type level-2-only
 net 49.0000.0000.0000.0006.00
 address-family ipv4 unicast
  metric-style wide  
 !
 interface BVI24
  bfd minimum-interval 300
  bfd multiplier 3
  bfd fast-detect ipv4
  point-to-point
  address-family ipv4 unicast
  !

bfd
 multipath include location 0/0/CPU0
!
```

### Verifying BFD over BVI

```
RP/0/RP0/CPU0:T-2006#show bfd all session 
Tue Jun 22 10:57:31.985 UTC

IPv4:
-----
Interface           Dest Addr           Local det time(int*mult)      State     
                                    Echo             Async   H/W   NPU     
------------------- --------------- ---------------- ---------------- ----------
BV24                192.6.17.17     0s(0s*0)         900ms(300ms*3)   UP        
                                                             Yes   0/0/CPU0     
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show bfd all session interface bvI 24 detail 

IPv4:
-----
<mark>I/f: BVI24, Location: 0/0/CPU0</mark>
Dest: 192.6.17.17
Src: 192.6.17.6
 <mark>State: UP for 0d:0h:3m:39s, number of times UP: 1
 Session type: SW/V4/SH/BR</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 12, your discr: 3, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 3, your discr: 12, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Label:
 Internal label: 24015/0x5dcf
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=5 ms, max=2750 ms, avg=1018 ms
       Last packet transmitted 218 s ago
   Rx: Number of intervals=6, min=5 ms, max=1446 ms, avg=384 ms
       Last packet received 218 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>MP download state: BFD_MP_DOWNLOAD_ACK</mark>
State change time: Jun 22 10:54:26.323
<mark>Session owner information:</mark>
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  isis-1               300 ms     3          300 ms     3         

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 16/9 

Platform Info:
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x3        Echo Session ID    : 0x0
Async Tx Key        : 0x3  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
</code>
</pre>
</div>

| Flags | Session Type                                                 |
|-------|--------------------------------------------------------------|
| SW    | Switched Session which includes BVI                          |
| V4    | IPv4 Session                                                 |
| SH    | Single Hop Session                                           |
| BR    | BFD Type is IRB                                              |

Again we can see the BFD over BVI interface is a single hop but a multipath session. 

Note: BFD over BVI for IPv4 and IPv6 is not supported yet on systems based on Jericho2 and NCS560.
{: .notice--info}

## BFD MultiHop Session

Till now we saw all the sessions be it Single-Path or Multi-Path, were Single-Hop. Now we will see Multi-Hop Multi-Path session. There are scenarios in which BFD needs to be enabled between two end-points which are not directly connected but across one or more Layer3 hops. For example BFD running over logical interfaces like GRE tunnel or protocol BGP. In such scenario path to reach other end-point will not always be same. As long as one path to the destination is active, BFD will come up. Also in such cases, there is not only one LC through which packet would leave the router and enter the router. And thus it is not just multi-hop but multi-path as well.   The encapsulation of BFD Control packets for multihop application in IPv4 and IPv6 is identical to that defined in BFD single hop, except that the UDP destination port must have a value of 4784.  This can aid in the demultiplexing and internal routing of incoming BFD packets.[[RFC 5883](https://datatracker.ietf.org/doc/html/rfc5883 "RFC 5883")]
We support BFD Multihop sessions for NCS5500 family and with the latest IOS-XR release 7.3.1 we have included other platforms from the NCS500 family as well. Let us verify the multihop BFD.



### Configuring the multi-hop BFD

```
router bgp 100
 bgp router-id 6.6.6.6
 address-family ipv4 unicast
neighbor 172.16.4.41
  remote-as 100
  bfd fast-detect
  bfd multiplier 3
  bfd minimum-interval 300
  update-source Loopback0
  address-family ipv4 unicast
  !
```

### Verifying the multi-hop BFD 


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:T-2006#show bfd all session detail</mark> 

IPv4:
-----

<mark>Location: 0/0/CPU0</mark>
Dest: 172.16.4.41
Src: 172.16.4.6
VRF Name/ID: default/0x60000000
 <mark>State: UP for 0d:0h:18m:4s, number of times UP: 1
 Session type: SW/V4/MH</mark>
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Multiplier: 3, diag: None
 My discr: 16, your discr: 4, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Multiplier: 3, diag: None
 My discr: 4, your discr: 16, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
async detection time: 900 ms(300 ms*3)
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=5 ms, max=1061 s, avg=354 s
       Last packet transmitted 1084 s ago
   Rx: Number of intervals=4, min=2 ms, max=302 ms, avg=152 ms
       Last packet received 1084 s ago
<mark>MP download state: BFD_MP_DOWNLOAD_ACK</mark>
State change time: Jun 22 11:58:19.853
Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  <mark>bgp-default </mark>         300 ms     3          300 ms     3         

<mark>H/W Offload Info:</mark>
 H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 0/0 

Platform Info:
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x4        Echo Session ID    : 0x0
Async Tx Key        : 0x4  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
          
</code>
</pre>
</div>

| Flags | Session Type                                                 |
|-------|--------------------------------------------------------------|
| SW    | Switched Session which includes BFD over Logical Bundle- BLB |
| V4    | IPv4 Session                                                 |
| MH    | Multi Hop Session                                            |

From the flags we can see the client BGP is having a **multi-hop** session. MP download state: BFD_MP_DOWNLOAD_ACK indicates that it is **multi-path** as well. 

## Timers and Scale

For the minimum timers and scale support, please visit the [BFD Architecture Document](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/).

## Reference

[CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/71x/b-routing-cg-ncs5500-71x/b-routing-cg-ncs5500-71x_chapter_0111.html#concept_hdq_sfm_gkb)
[RFC 5883](https://datatracker.ietf.org/doc/html/rfc5883 "RFC 5883")

## Summary

Hope this tech-note helped understanding the concept of BLB and its difference between [BoB](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/). We also touched upon the concept of Multi-Path and Multi-Hop sessions.
