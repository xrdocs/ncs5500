---
published: false
date: '2019-04-16 23:17 +0200'
title: NCS5500 QoS Part 2 - Verifying Buffering in Lab and Live Networks
author: Nicolas Fevrier
excerpt: >-
  Second part of the NCS5500 QoS Series: Packet Buffering illustrated in lab and
  in production networks
position: hidden
tags:
  - ncs5500
  - qos
  - production
  - buffers
---
{% include toc icon="table" title="NCS5500 Buffering Architecture Part 2" %}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, ACLs, Netflow following this [link](https://xrdocs.io/ncs5500/tutorials/).

Also you can find the first part of this post here:  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-1-understanding-packet-buffering/](https://xrdocs.io/ncs5500/tutorials/ncs5500-qos-part-1-understanding-packet-buffering/)  

## Checking Buffering in action

This second blog post will take concrete example to illustrate the concepts covered in the first part.   
The NCS5500 is based on a VOQ-only, single-lookup and ingress-buffering forwarding architecture.  
We will use a lab example to illustrate how the system handles burst, then we will present the monitoring tools / counters we can use to measure the numbers of packets handled in OCB or in DRAM, and finally we will present the data collected on 500+ NPUs in production.  
This should answer questions often asked by customers and clarify all potential doubts.

### Video

We recommend to start watching this short Youtube video first:  
[https://www.youtube.com/watch?v=1qXD70_cLK8](https://www.youtube.com/watch?v=1qXD70_cLK8)

<iframe width="560" height="315" src="https://www.youtube.com/embed/1qXD70_cLK8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

### Lab test

For this first part, and following customer request, we set up a large test bed:
- NCS5508 with two line cards 36x100G-A-SE (each card is made of 4 NPU Jericho+, each one handling 9 ports 100G)
- 27 tester ports 100GE (Spirent) connected to 27 router ports
	- 9 ports on LC 4 NPU 0
    - 9 ports on LC 4 NPU 1
    - 9 ports on LC 6 NPU 0

![]({{site.baseurl}}/images/testBed.jpg){: .align-center}  

We generate a background / constant traffic of 80% line rate (80Gbps on each port) between two NPUs. This traffic is illustrated in purple in the diagram above. Traffic is bi-directional.  
Then, we will use the remaining 9 ports to create burst traffic targeted to the ports Hu0/6/0/0-8 (shown in red in the diagram).  
The burst are 100ms every second.  

On the tester, we didn't make any specific PPM adjustment and used internal clock.

On the router itself, no specific configuration. Interfaces are configured with IPv4 addresses only and no QoS.

The test performed are the following:  
- test 1: 80% background and all the ports bursting at 20%. That means:
	- 900ms at 80% LR
    - 100ms at 100% LR
We verify no packets are dropped on the background or the burst, and we also make sure in the counters that packets are exclusely handled in the OCB.  
- test 2 : 
	- 80% background 
    - other ports bursting at 20%
    - one only bursts at 25%, create a 5Gbps saturation for 100ms
Here again, we verify no packets are dropped on the background or the burst, but also we verify that packets are sent to the DRAM. It's expected since one queue exceeds the threshold and is evicted to the external buffer.

This test is basic but has been asked by several customers who expected to see drops in such situations. It proves it's not the case, as designed.  

### Metrology

In the former test, we checked some counters to verify the behavior of the buffering.  
Let's review what's available and what should be used.  

We will collect the following Broadcom counters:  
- IQM_EnqueuePktCnt: total number of packets handled by the NPU
- IDR_MMU_CREDITS: total number of packets moved to DRAM
- IQM_EnqueueDscrdPktCnt: total number of packets dropped because of taildrop
- IQM_RejectDramIneligiblePktCnt: total number of packets dropped because DRAM was not accessible in read, typically when the bandwidth to DRAM is saturated
- and potentially also IDR_FullDramRejectPktsCnt and IDR_PartialDramRejectPktsCnt

Form CLI "show controller npu stats counters-all instance all location all" we can extract: ENQUEUE_PKT_CNT, MMU_IDR_PACKET_COUNTER and ENQ_DISCARDED_PACKET_COUNTER

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ROUTER#show controller npu stats counters-all instance all location all

FIA Statistics Rack: 0, Slot: 0, Asic instance: 0

Per Block Statistics:

Ingress:

NBI RX:
  RX_TOTAL_BYTE_COUNTER          = 161392268790033002
  RX_TOTAL_PKT_COUNTER           = 164628460653364

IRE:
  CPU_PACKET_COUNTER             = 0
  NIF_PACKET_COUNTER             = 164628460651867
  OAMP_PACKET_COUNTER            = 32771143
  OLP_PACKET_COUNTER             = 4787508
  RCY_PACKET_COUNTER             = 67452938
  IRE_FDT_INTRFACE_CNT           = 192

IDR:
  MMU_IDR_PACKET_COUNTER         = <mark>697231761913</mark>
  IDR_OCB_PACKET_COUNTER         = 1

IQM:
  ENQUEUE_PKT_CNT                = <mark>164640311902277</mark>
  DEQUEUE_PKT_CNT                = 164640311902198
  DELETED_PKT_CNT                = 0
  ENQ_DISCARDED_PACKET_COUNTER   = <mark>90015441</mark>
</code>
</pre>
</div>

To get the DRAM reject counters, we will use:  
- show contr npu stats counters-all detail instance all location all
or if the IOS XR version doesn't support the "detail" option, use the following instead:   
- show controllers fia diagshell 0 "diag counters" loc 0/x/CPU0

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ROUTER#show contr npu stats counters-all detail instance all location all | i Dram

  IDR FullDramRejectPktsCnt            :                0
  IDR FullDramRejectBytesCnt           :                0
  IDR PartialDramRejectPktsCnt         :                0
  IDR PartialDramRejectBytesCnt        :                0
  IQM0 RjctDramIneligiblePktCnt        :                0
  IQM1 RjctDramIneligiblePktCnt        :                0
  IDR FullDramRejectPktsCnt            :                0
  IDR FullDramRejectBytesCnt           :                0
  IDR PartialDramRejectPktsCnt         :                0
  IDR PartialDramRejectBytesCnt        :                0
  IQM0 RjctDramIneligiblePktCnt        :                0
  IQM1 RjctDramIneligiblePktCnt        :                0

--%--SNIP--%--SNIP--%--
  
</code>
</pre>
</div>

None of these counters are available through SNMP / MIB but instead you can use streaming telemetry:

From [https://github.com/YangModels/yang/blob/master/vendor/cisco/xr/653/Cisco-IOS-XR-fretta-bcm-dpa-hw-resources-oper-sub2.yang](https://github.com/YangModels/yang/blob/master/vendor/cisco/xr/653/Cisco-IOS-XR-fretta-bcm-dpa-hw-resources-oper-sub2.yang)
 
You'll found:
 
ENQUEUE_PKT_CNT: iqm-enqueue-pkt-cnt

    leaf iqm-enqueue-pkt-cnt {
      type uint64;
      description "Counts enqueued packets";
 
MMU_IDR_PACKET_COUNTER: idr-mmu-if-cnt

  leaf idr-mmu-if-cnt {
      type uint64;
      description
"Performance counter of the MMU interface";
 
ENQ_DISCARDED_PACKET_COUNTER: iqm-enq-discarded-pkt-cnt
 
leaf iqm-enq-discarded-pkt-cnt {
      type uint64;
      description
"Counts all packets discarded at the ENQ pipe";

At the moment (Apr 2019), RjctDramIneligiblePktCnt / FullDramRejectPktsCnt / PartialDramRejectPktsCnt are not available in the data model and can't be streamed.  

### Auditing real production routers

We have the counters available and we asked multiple customers (25+) to collect data from their production routers.  
In total, we had 550 NPUs transporting live traffic in multiple network positions:  
- IP core
- MPLS core (P/LSR)
- Internet border (transit / peering)
- CDN (connected to FB, Akamai, Google Cache, Netflix, ...)
- PE (L2VPN and L3VPN)
- Aggregation
- SPDC / ToR leaf

The data aggregated is helpful since it gives a vision of what is happening in reality.  
The total amount of traffic measured is tremendous: 24,526,679,839,376,100 packets!!!  
Not in lab, not in models, but in real routers.  

With the show commands described in former section, we extracted:  
- ENQUEUE_PKT_CNT: packets transmitted in the NPU
- MMU_IDR_PACKET_COUNTER: packets passed to DRAM
- ENQ_DISCARDED_PACKET_COUNTER: packets taildropped
- RjctDramIneligiblePktCnt: packets drop because of DRAM bandwidth

Dividing MMU_IDR_PACKET_COUNTER by ENQUEUE_PKT_CNT, we can compute the ratio of packets moved to DRAM.  
--> 0,151%  
This number is interesting. 

Dividing ENQ_DISCARDED_PACKET_COUNTER by ENQUEUE_PKT_CNT, we can compute the ratio of packets taildropped.  
--> 0,0358%  


Finally, RjctDramIneligiblePktCnt will tell us if we have situation in production where the link from the NPU to the DRAM gets saturated and drops packets.

### Recommendation for counter collection


