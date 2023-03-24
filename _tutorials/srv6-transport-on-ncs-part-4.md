---
published: true
date: '2023-03-23 16:37 +0530'
title: Segment Routing v6 (SRv6) Transport on NCS 5700
author: Paban Sarma
position: top
excerpt: >-
  SRv6 Transport on NCS 5700. Basic configuration approach comparison with NCS
  5500
tags:
  - iosxr
  - SRv6
  - Segment-Routing
  - NCS 5700
---


{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Deepak Balasubramanian, Technical Leader, Technical Marketing (deebalas@cisco.com)|  

<h2> Overview </h2>

Cisco MIG Access & Aggregation platforms, i.e NCS 500 and NCS 5500 series have variants built with BCM J1 and J2 ASICs. In our previous articles on SRv6 transport and services, we covered platforms built with first generation of BCM ASIC. While service configuration and behaviour are same on both the generations of platforms, there are specific config knobs needed on the first generation of platforms. We discussed the same in detail in our first article on [SRv6 Transport](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-1/). 

This tutorial will focus on various config knob differences to set up the SRv6 transport on the new generation of products i.e. the NCS 5700 and particular variants in NCS 540 series. 

<h2> Applicable Platform Models </h2>
  
 - NCS 500 Series  
    -- N540-24Q8L2DD-SYS
  - NCS 5500 Series  
    -- All NCS 57xx Fixed Platforms  
    -- NCS 5500 modular routers with NCS 57xx Line Cards in Native Mode

<h2> Differences in Configuration</h2>

### Platform hw-module command for format
To enable SRv6 transport on the NCS 5500 series (1st gen) we need to enable hw-module profile. This is not needed on the NCS 5700 series. SRv6 Mode base or uSID is configured directly under segment routing global configuration.

#### _NCS 5500 Configuration via hw-module_

<div class="highlighter-rouge">
<pre class="highlight">
<code>
hw-module profile segment-routing srv6 mode micro-segment format f3216
</code>
</pre>
</div>
    
#### _NCS 5700 Configuration_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
segment-routing
 srv6
  formats
   format usid-f3216
</code>
</pre>
</div>

### Defining traffic-class encapsulations
Another important factor in SRv6 is the traffic-class field in the encapsulated SRv6 header. The option is to either propagate from the payload or define a global value for all services. With NCS 5500 this is enabled along with the hw-module profile. On NCS 5700 this is configured under SRv6 encapsulation. The hw-module profile knob allows separate treatment for L2 and L3 encapsulation. 

#### _NCS 5500 Configuration via hw-module_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
hw-module profile segment-routing srv6 mode micro-segment format f3216
 <mark>encapsulation
  l2-traffic
   traffic-class propagate
  !
  l3-traffic
   traffic-class propagate</mark>
</code>
</pre>
</div>


#### _NCS 5700 Configuration_
<div class="highlighter-rouge">
<pre class="highlight">
<code>
segment-routing
 srv6
  <mark>encapsulation
   traffic-class propagate</mark>
  !
 !
!
</code>
</pre>
</div>

### Defining source encapsulation
The SRv6 header source address definition is done the same way on both NCS 5500 and NCS 5700 platform. This configuration is present under the segment-routing global configuration block.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
segment-routing
 srv6
  encapsulation
   <mark>source-address 2001::1</mark>
  !
 !
!
</code>
</pre>
</div>
### SRv6 Locator blocks
SRv6 locator is an important configuration parameter for SRv6. The locator is combination of Base and Node ID. For SRv6 uSID (f3216 format), the 32 bit base is divided into 24 bit BASE and 8 bit block ID. The following example shows configuration example from SRv6 locator for uSID.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
segment-routing
 srv6
  locators
   locator LOC
    micro-segment behavior unode psp-usd
   prefix fcbb:bb <mark>00</mark>:0001::/48
   !
  !
 !
!
</code>
</pre>
</div>
In the example above, the highlighted two nibbles can be in the range of _00-ff_ on NCS 5700 platforms. For the first generation, NCS 5500/500 the block ID range is _00-3f_.

### Service Configurations

As discussed in the Overview section, any type of service creation, show comamnds related to transport and service infra are common to all the NCS 500 and NCS 5500/5700 PIDs. The basic [Layer3](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/) and [Layer2](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-3/) service over SRv6 transport is already covered in our previous tutorials.

<h2> Summary </h2>
In this short article, we covered the fundamental difference in configuration approach for SRv6 transport between NCS 5700 and NCS 5500 platforms. Stay tuned for more SRv6 transport related content.
