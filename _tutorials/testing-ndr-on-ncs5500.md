---
published: false
date: '2019-09-23 17:22 +0200'
title: 'Testing NDR on NCS5500 36x 100GE Line Cards [Lab Series 02]'
author: Nicolas Fevrier
excerpt: Test to measure and explain the NDR on NCS5500 36x100G-SE line cards
position: hidden
tags:
  - ncs5500
  - lab series
  - testing
  - ndr
---
{% include toc icon="table" title="Testing NDR on 36x100G-SE" %}

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

This test is the second episode of our new Lab Series.  
You can find a detailed explanation on the purpose and also a link to all other tests in this xrdocs.io post:  [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)  
Last week, we run multiple tests in a very large test bed. Among the topics covered, we measured the NDR with a very long snake.  
The concept of Non Drop Rate deserves dedicated explanations.  
In this article and video, we will explain what it represents. We will demonstrate why the snake topology is not the best to reach the full capability of the ASIC and what happens when you push the system to its limit.

## Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/_i7xahJO1Yg?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## What is NDR?

The concept of Non Drop Rate, or NDR, is often used in system validation to qualify the NPU capability. It represents the minimum possible packet size you can transmit:
- on all ports of the NPU
- with 100% line rate on each port
- bi-directionally

Often times, it's something that can be derived with simple math starting from the number of packets per second the forwarding ASIC can support. But some other factors may come into play. 

We will see in these tests that a good understanding of the internal architecture may be necessary to interpret correctly the numbers measured in lab.

As useful it is to compare different devices, it's also very important to understand the limit of this number. NDR, like many other topics covered in this series of videos, is mostly a lab thing.  
In the example we are using today, the NDR for Jericho+ used in the 36x100G-SE line card, is 130 bytes per packet. For Jericho2, it will be around 230 or 280 bytes per packet depending on the line card. But it's virtually impossible to get 9 ports 100G (assigned to a same NPU) or worse, 12 ports 400G, running sustained line rate traffic directional and simultaneously.  
It's possible to imagine a couple of ports being saturated following a network outage, with a lot of traffic redirected via a specific path. But having the 9 ports in such situation reflects in a very poorly designed network, or... a lab situation.

**What about a DDoS attack trying to saturate the Jericho+ NPU?**

The question would have make sense 5 or 10 years ago, but it's not something that can be expected in current production network with such ASICs.  
We are talking about 900Gbps and 835MPPS here.
A DDoS attack larger than 900Gbps have been seen in the past, but the very nature of a DDoS is to be distributed (the first D in DDoS), that means the attack packets will come from everywhere on the internet and it will be virtually impossible to concentrate it on 9 ports of an NPU specifically. Attack packets will land naturally on many sites, many routers, many NPUs.  
With the same logic, if we can imagine attacks exceeding 835MPPS (with SYN flood, it has been seen in the past), the coordination of this attack on a specific NPU is extremely complex. And attackers with such compute power at their disposal will leverage other tools to attack their targets.

**Can we use a snake test for NDR measurement?**

Yes and no.  
Let's say it's not the best for such test, but considering the number of ports on the router, the tester interfaces necessary for a fully wired topology are extremely expensive and difficult to find.  
So the logical approach to load a line card or a chassis is to wire ports back-to-back and to configure VRFs, Bridging or MPLS static swapping. That way, we only need two ports from the traffic generator (actually one can be enough).  
But this methodology comes with a lot of limitations:
- TTL remarking may be needed for large setup
- latency can no longer be measured
- if numbers of ports are not consistent per NPU or per Core, it will affect the results
- a drop on a link or an NPU will be reflected on all the remaining ports: you can measure NDR with such topology but you can not trust any measurement below that level due to multiple cascading effects.

**Do I need a fully loaded chassis or even a fully loaded line card?**

Since we are testing the NPU capability, we just need to make sure the traffic is not locally routed/switched. That means we need to use at minimum two different NPUs and they to be all wired. So, with the line card we are using today, 9x2=18 ports would have been enough to run this test.  
Having more ports is impressive (and I admit, fun), but it doesn't not bring anything aside, may be, the power consumption measurement.

## Test results

In this test we were able to measure NDR at 130 bytes per packets. But also, we identified drops above this limit in some particular ranges.

### Under and above 130B/pkt



### Between 230B/pkt and 278B/pkt

### Performance per 100G ports

Test with 9 ports, asymmetrical port allocation

Test with just 2 ports at 64B and 256B

## Conclusion


----

CLI used during the test:

<div class="highlighter-rouge">
<pre class="highlight">
<code>monitor interface *
show controller fia diagshell 0 "diag counters g" location 0/0/CPU0
(admin) show controller fabric plane all statistics
show processes cpu
show interfaces hu0/0/0/0 accounting</code>
</pre>
</div>

![cpu.png]({{site.baseurl}}/images/cpu.png){: .align-center}




