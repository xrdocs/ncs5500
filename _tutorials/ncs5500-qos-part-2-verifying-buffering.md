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




at https://github.com/YangModels/yang/blob/master/vendor/cisco/xr/652/Cisco-IOS-XR-fretta-bcm-dpa-hw-resources-oper-sub2.yang
 
And found:
 
IQM_EnqueueDscrdPktCnt
 
    leaf iqm-enqueue-pkt-cnt {
      type uint64;
      description "Counts enqueued packets";
 
IQM_EnqueueDscrdPktCnt
 
leaf iqm-enq-discarded-pkt-cnt {
      type uint64;
      description
"Counts all packets discarded at the ENQ pipe";
 
IDR_MMU_CREDITS
 
  leaf idr-mmu-if-cnt {
      type uint64;
      description
"Performance counter of the MMU interface";
 

