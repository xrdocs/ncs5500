---
published: true
date: '2021-05-10 16:21 +0530'
title: Understanding the BFD Hardware Programming on NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document aims to cover the hardware programming of the BFD in the NCS55xx
  pipeline architecture. It also will cover the resource utilisation and  how to
  read and interpret the CLI outputs
tags:
  - NCS 5500
  - NCS 500
  - NCS5500
  - BFD
  - convergence
position: hidden
---
{% include toc icon="table" title="Understanding the BFD Hardware Programming on NCS55xx and NCS5xx" %} 

## Introduction

In our previous [artcile](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/), we discussed the BFD feature in the pipeline architecture (NCS55xx and NCS5xx). We discussed how the packet flow and the hardware resources are utilised. We saw how the scale is considered for the BFD feature and how well the resources have been carved to achieve the desired numbers. In this article, we will go a bit deeper. We will see a sample configuration, and see how to read the BFD outputs, hardware programming and resource utilization.

## Quick Refresh ([RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880))

As discussed in the previous article, the goal of Bidirectional Forwarding Detection (BFD) is to provide low-overhead, short-duration detection of failures in the path between adjacent forwarding engines, including the interfaces, data link(s), and, to the extent possible, the forwarding engines themselves. An additional goal is to provide a single mechanism that can be used for liveness detection over any media, at any protocol layer, with a wide range of Detection Times and overhead, to avoid a proliferation of different methods.

### BFD Control Packet Format

Let us look at the BFD control packet format briefly. This will be useful in understanding the CLI outputs and the messages being exchanged between 2 nodes. 

![Screenshot 2021-05-11 at 12.56.15 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-11 at 12.56.15 PM.png)

Apart from the above fields, we also have the optional Authentication section. For details please [refer](https://datatracker.ietf.org/doc/html/rfc5880).

| Field                         | Description                                                                                                                                                                                                                             |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Version (Vers)                | The version number of the protocol.                                                                                                                                                                                                     |
| Diagnostic (Diag)             | A diagnostic code specifying the local system's reason for the last change in session state. There are various values for the diagnostics which are defined in the RFC                                                                  |
| State (Sta)                   | This indicates the current BFD session state as seen by the transmitting system. The values are:<br>0 -- AdminDown<br>1 -- Down<br>2 -- Init<br>3 -- Up                                                                                 |
| Poll (P)                      | If set, the transmitting system is requesting verification of connectivity, or of a parameter change, and is expecting a packet with the Final (F) bit in reply.  <br>If clear, the transmitting system is not requesting verification. |
| Final (F)                     | If set, the transmitting system is responding to a received BFD Control packet that had the Poll (P) bit set.  <br>If clear, the transmitting system is not responding to a Poll.                                                       |
| Control Plane Independent (C) | If set, BFD is implemented in the forwarding plane and can continue to function through disruptions in the control plane).  <br>If clear, BFD implementation shares fate with its control plane.                                        |
| Authentication Present (A)    | If set, the Authentication Section is present and the session is to be authenticated.                                                                                                                                                   |
| Demand (D)                    | If set, Demand mode is active.<br>If clear, Demand mode is not active in the transmitting system.                                                                                                                                       |
| Multipoint (M)                | This bit is reserved for future point-to-multipoint extensions to BFD.                                                                                                                                                                  |
| Detect Mult                   | Detection time multiplier. The negotiated transmit interval, multiplied by this value, provides the Detection Time for the receiving system in Asynchronous mode.                                                                       |
| Length                        | Length of the BFD Control packet, in bytes.                                                                                                                                                                                             |
| My Discriminator              | A unique, nonzero discriminator value generated by the transmitting system, used to demultiplex multiple BFD sessions between the same pair of systems.                                                                                 |
| Your Discriminator            | The discriminator received from the corresponding remote system. This field reflects back the received value of My Discriminator, or is zero if that value is unknown.                                                                  |
| Desired Min TX Interval       | This is the minimum interval, in microseconds, that the local system would like to use when transmitting BFD Control packets.                                                                                                           |
| Required Min RX Interval      | This is the minimum interval, in microseconds, between received BFD Control packets that this system is capable of supporting.                                                                                                          |
| Required Min Echo RX Interval | This is the minimum interval, in microseconds, between received BFD Echo packets that this system is capable of supporting.                                                                                                              |

For detailed explaination of each field please [refer](https://datatracker.ietf.org/doc/html/rfc5880)


### BFD State Machine

![Screenshot 2021-05-11 at 2.24.16 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-11 at 2.24.16 PM.png)

As per the [RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880), the BFD state machine is quite straightforward.  There are three states through which a session normally proceeds: two for establishing a session : **Init and Up**. And one for tearing down a session: **Down**. This allows a three-way handshake for both session establishment and session teardown, assuring that both systems are aware of all session state changes.  A fourth state **AdminDown** exists so that a session can be administratively put down indefinitely. Each system communicates its session state in the State (Sta) field in the BFD Control packet.

| State           | Description                                                                                                                                                                                                                                                                                                                              |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Down state      | This state means that the session is down or has just been created. <br>A session remains in Down state until the remote system indicates that it agrees that the session is down by sending a BFD Control packet with the State field set to anything other than Up.                                                                    |
| Init state      | This state means that the remote system is communicating, and the local system desires to bring the session up, but the remote system does not yet realize it.  <br>A session will remain in Init state until either a BFD Control Packet is received that is signalling Init or Up state in which case the session advances to Up state |
| Up state        | This state means that the BFD session has successfully been established, and implies that connectivity between the systems is working. <br>The session will remain in the Up state until either connectivity fails or the session is taken down administratively.                                                                        |
| AdminDown state | This state means that the session is being held administratively down.  <br>This causes the remote system to enter Down state, and remain there until the local system exits AdminDown state                                                                                                                                             |

Note: For details on state machine please refer the RFC 5880 
{: .notice--info}


## Configuring a simple BFD session (NCS55xx and NCS5xx)

After a quick refresh of the theory behind the BFD packets, let us get into the routers and check it practically. We will take a simple example and walk through the hardware programming. We will take a very simple example and verify.

![Screenshot 2021-05-12 at 11.31.43 AM.png]({{site.baseurl}}/images/Screenshot 2021-05-12 at 11.31.43 AM.png)

We have configured OSPF between R1 and R2 and used BFD on the physical interface.

**R1 (NCS-55A2-MOD-HD-S)**

```
router ospf 1
 router-id 172.16.3.26
 address-family ipv4 unicast
 area 0
  !
  interface TenGigE0/0/0/12
   bfd minimum-interval 300
   bfd fast-detect
   bfd multiplier 3
   network point-to-point
  !
```

**R2 (N540-24Z8Q2C-M)**

```
router ospf 1
 router-id 172.16.3.18
 address-family ipv4 unicast
 area 0
  !
  interface TenGigE0/0/0/12
   bfd minimum-interval 300
   bfd fast-detect
   bfd multiplier 3
   network point-to-point
  !
```

## Verification in the Pipeline

Let us verify a few CLI commands and confirm the hardware programming. This command gives a quick information on the session configured in brief.

![Screenshot 2021-05-12 at 2.13.23 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-12 at 2.13.23 PM.png)

The below output gives a detailed output of the different parameters of the BFD control packet which we mentioned in the earlier section. We can see the source and destination values, the version, state, discriminator values and different flags being set or clear. We can also see the hardware offloaded information and values.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-26#show bfd ipv4 session interface tenGigE 0/0/0/12 detail 
<mark>I/f: TenGigE0/0/0/12, Location: 0/0/CPU0
Dest: 192.18.26.18
Src: 192.18.26.26</mark>
 State: UP for 0d:2h:6m:39s, number of times UP: 1
 Session type: PR/V4/SH
<mark>Received parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147487749, your discr: 2147491924, state UP, D/F/P/C/A: 0/0/0/1/0
<mark>Transmitted parameters:</mark>
 Version: 1, desired tx interval: 300 ms, required rx interval: 300 ms
 Required echo rx interval: 0 ms, multiplier: 3, diag: None
 My discr: 2147491924, your discr: 2147487749, state UP, D/F/P/C/A: 0/1/0/1/0
Timer Values:
 Local negotiated async tx interval: 300 ms
 Remote negotiated async tx interval: 300 ms
 Desired echo tx interval: 0 s, local negotiated echo tx interval: 0 ms
 Echo detection time: 0 ms(0 ms*3), async detection time: 900 ms(300 ms*3)
Local Stats:
 Intervals between async packets:
   Tx: Number of intervals=3, min=7 ms, max=2184 ms, avg=829 ms
       Last packet transmitted 7598 s ago
   Rx: Number of intervals=6, min=4 ms, max=1700 ms, avg=461 ms
       Last packet received 7598 s ago
 Intervals between echo packets:
   Tx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet transmitted 0 s ago
   Rx: Number of intervals=0, min=0 s, max=0 s, avg=0 s
       Last packet received 0 s ago
 Latency of echo packets (time between tx and rx):
   Number of packets: 0, min=0 ms, max=0 ms, avg=0 ms
<mark>Session owner information:
                            Desired               Adjusted
  Client               Interval   Multiplier Interval   Multiplier
  -------------------- --------------------- ---------------------
  ospf-1               300 ms     3          300 ms     3 </mark>        

<mark>H/W Offload Info:
 H/W Offload capability : Y, Hosted NPU     : 0/0/CPU0</mark>
 Async Offloaded        : Y, Echo Offloaded : N
 Async rx/tx            : 292/45 

<mark>Platform Info:</mark>
NPU ID: 0 
Async RTC ID        : 1          Echo RTC ID        : 0
Async Feature Mask  : 0x0        Echo Feature Mask  : 0x0
Async Session ID    : 0x2054     Echo Session ID    : 0x0
Async Tx Key        : 0x80002054  Echo Tx Key        : 0x0
Async Tx Stats addr : 0x0   Echo Tx Stats addr : 0x0
Async Rx Stats addr : 0x0   Echo Rx Stats addr : 0x0
</code>
</pre>
</div>

Note: Session type: PR/V4/SH.    
Here PR = Pre-routed session (eg: sessions bound to an interface like pos, serial, etc  
If its SW = Switched session (eg: BLB, BFD over TE, Multi-hop sessions)  
V4 = IPv4  
SH = Single Hop  
{: .notice--info}


Let us check the hardware programming for the discriminator values. This is the most important thing to check, if peer and local routers are exchanging each others discriminator values. In the below output, **session_handle** should match **our discriminator**


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-26#show dpa objects bfdhwoff location 0/0/CPU0
  ofa_npu_mask_t npu_mask => 1 
  uint8 unit => 0
  uint8 slot => 0
  uint16 xid => 1
 <mark>@uint32 session_handle => 2147491924</mark>
 @uint16 soft_session_handle => 0
  uint32 flags => 8
  int id => (not set)
  uint8_t remote_id_grid_alloc => 1
  uint32_t alloc_sz => (not set)
  uint32_t remote_id => 1
  <mark>ofa_encap_id_t tunnel_encapid => 0x4001380b</mark>
  uint8_t tunnel_encapid_grid_alloc => 1
  uint8_t remote_id_rid_level => 0
  uint8_t tunnel_encapid_rid_level => 1
  uint8 type => (not set)
  int tx_gport => (not set)
  int remote_gport => (not set)
  uint16 dest_port => 40
  int bfd_period => 300
  uint16 vpn => (not set)
  uint8 vlan_pri => (not set)
  uint8 inner_vlan_pri => (not set)
  uint32 vrf_id => 3758096384
  uint32 egress_label => (not set)
  uint8 egress_ttl => (not set)
  uint8 egress_exp => (not set)
  uint32 label => (not set)
  <mark>uint32 egress_if => 1073807360</mark>
  uint32 secondary_egress_if => (not set)
  uint32 egress_ifh => 320
  uint32 secondary_egress_ifh => (not set)
  uint8 primary_tunnel_bit_index => (not set)
  uint32 bundle_ifh => (not set)
  uint8 mep_id => (not set)
  uint8 mep_id_length => (not set)
  int int_pri => 24
  uint8 cpu_qid => (not set)
  uint8 local_state => 3
  <mark>uint32 local_discr => 2147491924</mark>
  uint32 local_min_tx => 300000
  uint32 local_min_rx => 300000
  uint8 local_detect_mult => 3
  uint32 bfd_detection_time => 900000
  <mark>uint32 remote_discr => 2147487749</mark>
  uint32 remote_flags => 0
  uint8 remote_state => 0
  uint32 bfd_session_type => 0
  uint8 ip_tos => 192
  uint8 ip_ttl => 255
  uint32 udp_src_port => 49153
  uint32 dst_ip_addr => 3222411794
  dpa_ip6_addr_t dst_ip6_addr => (not set)
  uint32 src_ip_addr => 3222411802
</code>
</pre>
</div>

Note: This is trimmed output 
{: .notice--info}

From the **tunnel_encapid** take last 5 digits and run the below command. We get the value of  **Next_eep**. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-26#show controller fia diagshell 0 "diag pp lif type=out id=0x1380b gl=1" location 0/0/cpu0</mark>
Node ID: 0/0/CPU0
************************************************ 
Warning: Core 0: Packet diagnostics refers to OLP packet 
************************************************ 
Global Out_LIF:0x0001380b -> Local Out_LIF:0x0000e0ca -> Type:(null) Bank:6 Offset:101
DATA Encapsulation:
data_entry[0]: 0x121a1a0c data_entry[1]: 0x0006a8c0 data_entry[2]: 0x00000000 oam_lif_set: 0
<mark>Next_eep: 0x600a</mark>
RP/0/RP0/CPU0:N55-26#
</code>
</pre>
</div>

Convert the egress_int from decimal to hex. And then use the last 5 digits and run the below command. The **Local Out_LIF** should match the **Next_eep**. We can also verify the **dest_mac** which can be verified by the show arp command

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-26#show controller fia diagshell 0 "diag pp lif type=out id=0x10000 gl=1" location 0/0/cpu0</mark>
Node ID: 0/0/CPU0
************************************************ 
Warning: Core 0: Packet diagnostics refers to OLP packet 
************************************************ 
Global Out_LIF:0x00010000 -> <mark>Local Out_LIF:0x0000600a</mark> -> Type:(null) Bank:2 Offset:5
LL Encapsulation:
<mark>dest_mac:00:32:17:80:98:30</mark>
out_vid_valid: 1
out_vid: 19
pcp_dei_valid: 0
pcp_dei: 0
tpid_index: 0
ll_remark_profile: 0
out_ac_valid: 0
out_ac_lsb: 0
oam_lif_set: 0
outlif_profile: 0x10
is native: 0
Next_eep: 0xffffffff
RP/0/RP0/CPU0:N55-26#
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-26#show arp 192.18.26.18</mark>
-------------------------------------------------------------------------------
0/0/CPU0
-------------------------------------------------------------------------------
Address         Age        Hardware Addr   State      Type  Interface
192.18.26.18    00:27:14   <mark>0032.1780.9830</mark>  Dynamic    ARPA  TenGigE0/0/0/12
RP/0/RP0/CPU0:N55-26#
</code>
</pre>
</div>

From the below output we can see the properties of the BFD. It clearly mentions that the BFD is processed in the OAM engine and not in the CPU. Other hardware values programmed are also matching.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-26#show controllers fia diagshell 0 "diag oam ep id=0x2054 location 0/0/CPU0</mark> 
Node ID: 0/0/CPU0
<mark>=====BFD endpoint ID: 0X2054</mark> 
        ***Properties:
        <mark>Type: BFD over IPV4</mark>
        <mark>BFD session is processed in OAMA (rather than CPU)</mark>. TX gport: 0X6c000028
        Remote gport: 0X160040e3
        gport disabled
        Source address: 0.0.0.0 <mark>UDP source port: 49153</mark>
        <mark>Local state: up,        Remote state: up</mark>
        Local diagnostic: No Diagnostic,        Remote diagnostic: No Diagnostic
        Remote Flags: 0x8,       Local Flags 0x8
        Queing priority: 24
        BFD rate (ms): 301
        Desired local min TX interval: 300000, Required local RX interval: 300000
        Local discriminator: 2147491924, Local detection multiplier: 3
        <mark>Remote discriminator: 2147487749</mark>
        Remote detection explicit timeout: 899300
RP/0/RP0/CPU0:N55-26#
</code>
</pre>
</div>

## Packet Captures

Let us examine the packet capture of the control packets being exchanged between the routers. Below is the packet capture of the received parameters from the remote peer. We can verify that the values are matching with the CLI outputs we captured in the earlier commands

![Screenshot 2021-05-12 at 4.54.22 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-12 at 4.54.22 PM.png)

## QoS treatment for BFD packets

From the below output we can see the BFD control packets are marked by default with highest priority i.e TC7

```
class-map match-any BFD
 match traffic-class 7 
 end-class-map
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-26#show policy-map interface tenGigE 0/0/0/12        
TenGigE0/0/0/12 output: BFD
Class BFD
  Classification statistics          (packets/bytes)     (rate - kbps)
    <mark>Matched             :                 914/97196                1</mark>
    <mark>Transmitted         :                 914/97196                1</mark>
    Total Dropped       :                   0/0                    0
  Queueing statistics
    Queue ID                             : 1127 
    Taildropped(packets/bytes)           : 0/0
Class class-default
  Classification statistics          (packets/bytes)     (rate - kbps)
    Matched             :                 105/8204                 0
    Transmitted         :                 105/8204                 0
    Total Dropped       :                   0/0                    0
  Queueing statistics
    Queue ID                             : 1120 
    Taildropped(packets/bytes)           : 0/0
</code>
</pre>
</div>

## Reference

- [ASR9k BFD Implementation](https://community.cisco.com/t5/service-providers-documents/bfd-support-on-cisco-asr9000/ta-p/3153191)
- [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/routing/73x/b-routing-cg-ncs5500-73x/implementing-bfd.html)

## Summary

Hope this article helped you to understand the hardware programming of the BFD in the pipeline architecture. This can be used for a basic debugging before reaching out to Cisco TAC :). In the next arcticle we will try to explore the concepts of BLB/BOB/BVI and its hardware implementation.
