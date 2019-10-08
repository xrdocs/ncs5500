---
published: true
date: '2019-10-03 08:52 +0200'
title: 'NCS5500 Fabric Redundancy Tests [Lab Series 03] '
author: Nicolas Fevrier
excerpt: How an NCS5500 modular chassis behaves with 5 out of 6 fabric cards?
tags:
  - ncs5500
  - fabric
  - redundancy
  - lab
  - test
position: top
---
{% include toc icon="table" title="NCS5500 Fabric Redundancy Tests" %} 

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Third episode of the lab series, today, we will talk about NCS5500 fabric. More specifically, we will try to qualify the impact of losing a fabric card in a chassis.  
The goal of these blog posts is to describe tests performed in lab, detail the methodology and the results, and finally provide additional information on the internals of the NCS5500 platforms.  
All the former tests are listed here: [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)

## Video

In the video below, we are showing a test with a 36x100G-SE line card used with 6 fabric cards then 5.

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/SZHuq7d2QOM?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## Architecture

As you certainly know, the NCS5500 exists in fixed systems and modular chassis. For this second category, we have 4-slot, 8-slot and 16-slot versions.  

![3-chassis.png]({{site.baseurl}}/images/3-chassis.png){: .align-center}


If they can all mix and match different type of line cards, each chassis type is using specific fabric cards. They have different size, which is expected considering the orthogonal design where all line cards are directly plugged into the fabric cards.

![archi.png]({{site.baseurl}}/images/archi.png){: .align-center}

Also, we have now two generations of fabric cards.  
The v1 supports line cards equipped with Jericho and Jericho+ NPUs. The v2 are supporting the same line cards but also the new ones powered by Jericho2 ASICs.  

Depending on the chassis size, each fabric card will be made of one or multiple Fabric Engine:  

In the first generation fabric cards, we have one or several FE3600 ASICs:

![SFE-v1.png]({{site.baseurl}}/images/SFE-v1.png){: .align-center}

In the second generation, we will use a new fabric engine named "Ramon" (or FE9600)

![SFE-v2.png]({{site.baseurl}}/images/SFE-v2.png){: .align-center}

In summary, per Fabric Card:

| Fabric Engines | v1 | v2 |
|:-----:|:-----:|:-----:|
| NCS5504 | 1 | 1 |
| NCS5508 | 2 | 2 |
| NCS5516 | 6 | 3 |

Regardless of the generation, each router is using 6 fabric cards. They can operate with less than 6 but it could be at the expense of the bandwidth of some line cards. That's what we will detail in this blog post.

## Theory

Now, let's present the math used to identify the bandwidth available when losing one fabric card and how it affects each type of line cards in our portfolio.

Each Jericho NPU connects to each Fabric Module with 6 SERDES at 25Gbps raw: after cell tax, encoding and correction, we can actually use 20.8Gbps. Each Jericho+ NPU connects to each Fabric Module with 8 SERDES links, with the same useable bandwidth.

### 4-slot and 8-slot Chassis with FE3600 fabric and Jericho Line Cards

4-slot: From a Jericho NPU's perspective, all 6 links are connected to a single FE3600 per fabric card:

![4slot-Jericho.png]({{site.baseurl}}/images/4slot-Jericho.png){: .align-center}

8-slot: Similarly, the 6 links from each Jericho NPU are split in 3+3 to the two FE3600s:

![8slot-Jericho.png]({{site.baseurl}}/images/8slot-Jericho.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 6x 25Gbps x 6FM = 900Gbps (raw)
- 6x 20.8Gbps x 6FM = 748Gbps

With only 5 Fabric Modules:
- 6x 25Gbps x 5FM = 750Gbps (raw)
- 6x 20.8Gbps x 5FM = 624Gbps 

Now, if we consider the bandwidth required per NPU for the different Jericho line cards, we can see how much the fabric will be able to accomodate in case of fabric loss:

| Line Card | Ports per Jericho NPU (Gbps) | <624G? |
|:-----:|:-----:|:-----:|
| 36x100G | 600 | Yes |
| 36x100G-S | 600 | Yes |
| 24x100G-SE | 600 | Yes |
| 24H12F | 720 | No |
| 18H12F | 840 (but ASIC allows 720) | No |
| 6x200G-COH | 600 | Yes |

### 4-slot and 8-slot Chassis with FE3600 fabric and Jericho+ Line Cards

4-slot: from the Jericho+ NPU's perspective, all 8 links are connected to a single FE3600 per fabric card:

![4slot-Jplus.png]({{site.baseurl}}/images/4slot-Jplus.png){: .align-center}

8-slot: the 8 links from each Jericho+ NPU are split in 4+4 to the two FE3600s:

![8slot-Jplus.png]({{site.baseurl}}/images/8slot-Jplus.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 8x 25Gbps x 6FM = 1200Gbps (raw)
- 8x 20.8Gbps x 6FM = 998Gbps

With only 5 Fabric Modules:
- 8x 25Gbps x 5FM = 900Gbps (raw)
- 8x 20.8Gbps x 5FM = 832Gbps 

Here again, let's check the bandwidth necessary for line rate with Jericho+ line cards:

| Line Card | Ports per J+ NPU (Gbps) | <832G? |
|:-----:|:-----:|:-----:|
| 36x100G-SE | 900 | No |
| MOD-A | 1,000 (but ASIC allows 900) | No |
| MOD-A-SE | 1,000 (but ASIC allows 900) | No |

Now mixing Jericho and Jericho+ in the same chassis:

|  |  | 6 Fabrics | 5 Fabrics |
|:-----:|:-----:|:-----:|:-----:|
| J@600G | J@600G | 100% 600G | 100% 600G |
| J@720G | J@720G | 100% 720G | 87% 624G |
| J | J+ | 100% 600G | 100% 600G |
| J+ | J+ | 100% 900G | 92% 828G |

### 16-slot Chassis with FE3600 fabric and Jericho Line Cards

Each Jericho ASIC has 6 links at 25Gbps, that are equally distributed to the 6 FE3600 of each fabric card:

![16slot-J.png]({{site.baseurl}}/images/16slot-J.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 6x 25Gbps x 6FM = 900Gbps (raw)
- 6x 20.8Gbps x 6FM = 748Gbps

With only 5 Fabric Modules:
- 6x 25Gbps x 5FM = 750Gbps (raw)
- 6x 20.8Gbps x 5FM = 624Gbps 

Same logic and results than 4-slot and 8-slot chassis.  
How much the fabric will be able to accomodate in case of fabric loss?

| Line Card | Ports per Jericho NPU (Gbps) | <624G? |
|:-----:|:-----:|:-----:|
| 36x100G | 600 | Yes |
| 36x100G-S | 600 | Yes |
| 24x100G-SE | 600 | Yes |
| 24H12F | 720 | No |
| 18H12F | 840 (but ASIC allows 720) | No |
| 6x200G-COH | 600 | Yes |

### 16-slot Chassis with FE3600 fabric and Jericho+ Line Cards

Things are getting a bit more complex when we use Jericho+ NPUs in 16-slot chassis. Indeed, each NPU has 8 SERDES (links at 25Gbps raw bw) and they need to connect to fabric cards made of 6 FE3600, we need to address an unequal distribution:

![16-slot-Jplus-1.png]({{site.baseurl}}/images/16-slot-Jplus-1.png){: .align-center}

We will have some two FE3600 connected with 2 SERDES links while the rest will have only one connection.

So as long as we have a flow transiting from NPU X of line A to NPU X of line B, we don't have issues (example: Jericho+ instance 0 of LC0 pushing traffic to J+ instance 0 in LC1):

![16-slot-Jplus-2.png]({{site.baseurl}}/images/16-slot-Jplus-2.png){: .align-center}

The situation is less ideal when the communication is from NPU X of line A to NPU Y of line B, we have paths where two links can only be used at the bandwidth of one:

![16-slot-Jplus-3.png]({{site.baseurl}}/images/16-slot-Jplus-3.png){: .align-center}

In this example above (Jericho+ 0 on LC0 pushing traffic to NPU 1 on LC1), we can't get line rate traffic because we have only 40 SERDES links.  
Let's take this example for the math below:

In nominal state, with 6 Fabric Modules:
- 40x 25Gbps = 1000Gbps (raw)
- 40x 20.8Gbps = 832Gbps

That's indeed below the 900Gbps of bandwidth capability of a Jericho+ ASIC, in nominal state (note it's a worst case situation, remember that the traffic targeted to the same ASICs are not transiting through the fabric and that aligned NPUs (NPU X to NPU X in a different line card) have full bandwidth available).
{: .notice-info}

With only 5 Fabric Modules:
- 657Gbps

The bandwidth necessary for line rate with Jericho+ line cards:

| Line Card | Ports per J+ NPU (Gbps) | <657G or 832G? |
|:-----:|:-----:|:-----:|
| 36x100G-SE | 900 | No |
| MOD-A | 1,000 (but ASIC allows 900) | No |
| MOD-A-SE | 1,000 (but ASIC allows 900) | No |

Note: this bandwidth problem is addressed with the second generation fabric cards on the 16-slot chassis.
{: .notice-info}

Finally, mixing Jericho and Jericho+ in the same chassis:

|  |  | 6 Fabrics | 5 Fabrics |
|:-----:|:-----:|:-----:|:-----:|
| J | J | 100% 600G | 100% 600G |
| J | J+ | 100% 600G | 100% 600G |
| J+ | J | 82% 738G | 69% 621G |
| J+ | J+ | 92% 828G | 73% 657G |

### 8-slot Chassis with Ramon/FE9600 fabric and Jericho Line Cards

At the moment of this blog publication, the 4-slot chassis doesn't support the next generation fabric cards, it's still in the roadmap. But all the principles and results shared below can also be used for the 4-slot.

All 6 links from the Jericho ASICs are split in 3+3 on the two Ramon Fabric Engines in each fabric card.

![8-slot-Ramon-J.png]({{site.baseurl}}/images/8-slot-Ramon-J.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 6x 25Gbps x 6FM = 900Gbps (raw)
- 6x 20.8Gbps x 6FM = 748Gbps

With only 5 Fabric Modules:
- 6x 25Gbps x 5FM = 750Gbps (raw)
- 6x 20.8Gbps x 5FM = 624Gbps 

How much the fabric will be able to accomodate in case of fabric loss?

| Line Card | Ports per Jericho NPU (Gbps) | <624G? |
|:-----:|:-----:|:-----:|
| 36x100G | 600 | Yes |
| 36x100G-S | 600 | Yes |
| 24x100G-SE | 600 | Yes |
| 24H12F | 720 | No |
| 18H12F | 840 (but ASIC allows 720) | No |
| 6x200G-COH | 600 | Yes |

Note: these results are the same with the FE3600. It's expected since new fabric cards don't change the way the NPUs are attached to them.
{: .notice-info}

### 8-slot Chassis with Ramon/FE9600 fabric and Jericho+ Line Cards

The 8 links from the Jericho+ NPUs are split in 4+4 on the two Ramon Fabric Engines in each fabric card.

![8-slot-Ramon-Jplus.png]({{site.baseurl}}/images/8-slot-Ramon-Jplus.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 8x 25Gbps x 6FM = 1,200Gbps (raw)
- 8x 20.8Gbps x 6FM = 998Gbps

With only 5 Fabric Modules:
- 8x 25Gbps x 5FM = 1,000Gbps (raw)
- 8x 20.8Gbps x 5FM = 832Gbps 

The bandwidth necessary for line rate with Jericho+ line cards:

| Line Card | Ports per J+ NPU (Gbps) | <657G or 832G? |
|:-----:|:-----:|:-----:|
| 36x100G-SE | 900 | No |
| MOD-A | 1,000 (but ASIC allows 900) | No |
| MOD-A-SE | 1,000 (but ASIC allows 900) | No |

Here again, no difference for the 8-slot chassis.
{: .notice-info}

### 8-slot Chassis with Ramon/FE9600 fabric and Jericho2 Line Cards

![8-slot-Ramon-J2.png]({{site.baseurl}}/images/8-slot-Ramon-J2.png){: .align-center}

In nominal state, with 6 Fabric Modules:
- 18x 53.125Gbps x 6FM = 5,737Gbps (raw)
- 18x 45.8Gbps x 6FM = 4,946Gbps

With only 5 Fabric Modules:
- 18x 53.125Gbps x 5FM = 4781Gbps (raw)
- 18x 45.8Gbps x 5FM = 4122Gbps (85.8% of 4800Gbps)

At the moment of this publication, we have two line cards based on Jericho-2:  
NC55-24DD where each J2 services 4.8Tbps  
and NC55-18DD-SE where each J2 services 3.6Tbps.

| Line Card | Ports per J2 NPU (Gbps) | <4,122G |
|:-----:|:-----:|:-----:|
| 24x400G | 4,800 | No |
| 18x400G-SE | 3,600 | Yes |

### 16-slot Chassis with Ramon/FE9600 fabric and Jericho Line Cards

![16-slot-Ramon-J.png]({{site.baseurl}}/images/16-slot-Ramon-J.png){: .align-center}

Same math and behavior than Jericho line cards with 8-slot

| Line Card | Ports per Jericho NPU (Gbps) | <624G? |
|:-----:|:-----:|:-----:|
| 36x100G | 600 | Yes |
| 36x100G-S | 600 | Yes |
| 24x100G-SE | 600 | Yes |
| 24H12F | 720 | No |
| 18H12F | 840 (but ASIC allows 720) | No |
| 6x200G-COH | 600 | Yes |

### 16-slot Chassis with Ramon/FE9600 fabric and Jericho+ Line Cards

![16-slot-Ramon-Jplus.png]({{site.baseurl}}/images/16-slot-Ramon-Jplus.png){: .align-center}

The limitation encountered on the FE3600-based fabric cards on the 16-slot disappears with the new generation fabric cards.  
The support of line cards and impact of a fabric card loss is the same than 8-slot:

In nominal state, with 6 Fabric Modules:
- 8x 25Gbps x 6FM = 1,200Gbps (raw)
- 8x 20.8Gbps x 6FM = 998Gbps

With only 5 Fabric Modules:
- 8x 25Gbps x 5FM = 1,000Gbps (raw)
- 8x 20.8Gbps x 5FM = 832Gbps 

| Line Card | Ports per J+ NPU (Gbps) | <998G? | <832G? |
|:-----:|:-----:|:-----:|:-----:|
| 36x100G-SE | 900 | Yes | No |
| MOD-A | 1,000 (but ASIC allows 900) | Yes | No |
| MOD-A-SE | 1,000 (but ASIC allows 900) | Yes | No |


### 16-slot Chassis with Ramon/FE9600 fabric and Jericho2 Line Cards

![16-slot-Ramon-J2.png]({{site.baseurl}}/images/16-slot-Ramon-J2.png){: .align-center}

Same behavior and results than 8-slot.

| Line Card | Ports per J2 NPU (Gbps) | <4,946G |
|:-----:|:-----:|:-----:|
| 24x400G | 4,800 | No |
| 18x400G-SE | 3,600 | Yes |

## Test results

Now that we have all the theory clarified, let's see what has been tested and measured in this video.  
We are using:
- an 8-slot chassis
- FE3600 fabric cards (first generation)
- 36x100G-SE line cards (based on 4x Jericho+ NPUs)

To simplify the process (and not remove the fan trays), we shut down the fabric cards electrically from the admin config level:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS5508-2#admin
root connected from 127.0.0.1 using console on NCS5508-2
sysadmin-vm:0_RP0# hw-module location 0/fc0 ?
Possible completions:
  offline    Take a hardware module offline for diagnostics
  online     Take a hardware module online for normal operation
  reload     Reload a hardware module
  shutdown   Shut down a hardware module
sysadmin-vm:0_RP0# hw-module location 0/fc0 shut
Shutdown hardware module ? [no,yes] yes
result Card graceful shutdown request on 0/FC0 succeeded.
sysadmin-vm:0_RP0# sh platfo
Location  Card Type               HW State      SW State      Config State
----------------------------------------------------------------------------
0/0       NC55-36X100G            OPERATIONAL   OPERATIONAL   NSHUT
0/6       NC55-36X100G-A-SE       OPERATIONAL   OPERATIONAL   NSHUT
0/RP0     NC55-RP-E               OPERATIONAL   OPERATIONAL   NSHUT
0/RP1     NC55-RP-E               OPERATIONAL   OPERATIONAL   NSHUT
0/FC0     NC55-5508-FC            <mark>POWERED_OFF</mark>   <mark>SW_INACTIVE</mark>   NSHUT
0/FC1     NC55-5508-FC            OPERATIONAL   OPERATIONAL   NSHUT
0/FC2     NC55-5508-FC            OPERATIONAL   OPERATIONAL   NSHUT
0/FC3     NC55-5508-FC            OPERATIONAL   OPERATIONAL   NSHUT
0/FC4     NC55-5508-FC            OPERATIONAL   OPERATIONAL   NSHUT
0/FC5     NC55-5508-FC            OPERATIONAL   OPERATIONAL   NSHUT
0/FT0     NC55-5508-FAN           OPERATIONAL   N/A           NSHUT
0/FT1     NC55-5508-FAN           OPERATIONAL   N/A           NSHUT</code>
</pre>
</div>

We use a snake topology (with the limits described in the former posts), but limited to a single line card, 36 ports 100GE.

Based on the math above, we should get around 830Gbps of forwarding capacity when running on five fabric cards, that's 92% of the nominal mode. Logically, we could expect to see 8% loss on the traffic generator.

For 1500B:

![1500B.png]({{site.baseurl}}/images/1500B.png){: .align-center}

We measure a bit more than 11% loss.

For 500B:

![500B.png]({{site.baseurl}}/images/500B.png){: .align-center}

We measure 8 to 10% loss.

For 130B:

![130B.png]({{site.baseurl}}/images/130B.png){: .align-center}

We measure 12% to 16% loss.

**How can we explain this deviation from the theory?**

It's actually exactly the same behavior than what we explain in the previous blog about NDR.  
[https://xrdocs.io/ncs5500/tutorials/testing-ndr-on-ncs5500/](https://xrdocs.io/ncs5500/tutorials/testing-ndr-on-ncs5500/)  
In this blog post, we described a case of fabric saturation, generating a backpressure message for the particular VOQ, triggering the eviction of the queue to the DRAM and eventually saturating the DRAM bandwidth (because the saturation was maintained along multiple seconds).  
In the lab, we verify that indeed we have counters pointing in this direction.

![shcontr.png]({{site.baseurl}}/images/shcontr.png){: .align-center}

So we are no longer measuring the impact of the fabric loss, but we are also forcing so much traffic that it exceeds the bandwidth to the deep buffer.

It's illustrated by the "Rejects" reasons in the ENQ_DISCARDED_PACKET_COUNTER above.

## Next one ?

We will demonstrate the FIB scale on the large external TCAM systems. If you are interested in any other test case, don't hesitate to let us know in the blog notes.
