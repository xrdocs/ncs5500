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

### 