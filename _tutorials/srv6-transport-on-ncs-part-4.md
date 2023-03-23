---
published: true
date: '2022-11-14 15:25 +0530'
title: SRv6 Transport on NCS 5700
position: hidden
author: Paban Sarma
excerpt: >-
  SRv6 Transport building with NCS 5700. compare the difference in configuration
  option w.r.t NCS 5500
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  

<h2> Overview </h2>

Cisco MIG Access & Aggreagtion product line, i.e NCS 500 and NCS 5500 series has variants built with BCM J1 and J2 ASICs. Our previous articles, on SRv6 transport and Services on top of it covered platforms built with first generation of BCM ASIC. While, service configuration and behaviour are same on both generation, there are specific config knobs needed on the first generation of platforms. we discussed the same in details in our first article on [SRv6 Transport](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-1/). This is particularly, because Cisco being the only vendor that support SRv6 transport on first generation of BCM ASIC using custom built SDKs. 

This article, will focus on config knob differences to setup the SRv6 transport on new generation of products i.e. the NCS 5700 and particular variants in NCS 540 series. 

<h2> Applicable Platform Models </h2>
  
 - NCS 500 Series
  	-- put Arches PID
  - NCS 5500 Series
  	-- Put NCS 5700 Fixed PIDs
  	-- NCS 5500 modular routers running native-mode (with NCS 5700 LC ) 

<h2> Differences in Configuration</h2>

### Platform hw-module command for format
To enable, SRv6 transport on the NCS 5500 series (1st gen) we need to enable hw-module profile. This is not needed on the NCS 5700 series. SRv6 Mode base or uSID is configured directly under segment routing global configuration.

#### NCS 5500 Configuration via hw-module

<div class="highlighter-rouge">
<pre class="highlight">
<code>
hw-module profile segment-routing srv6 mode micro-segment format f3216
</code>
</pre>
</div>
	
#### NCS 5700 Configuration
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
Another important factor in SRv6 is the traffic-class filed in the encapsulated SRv6 header. The otion is to either propagate from the payload or define a global value for all services. With NCS 5500 this is enabled along with the hw-module profile. While for NCS 5700 this is configured under SRv6 encapsulation. The hw-module profile allows a knob separately treat for l2 and l3 encapsulation. 

#### NCS 5500 Configuration via hw-module
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

#### NCS 5700 Configuration

#### NCS 5700 Configuration
<div class="highlighter-rouge">
<pre class="highlight">
<code>
segment-routing
 srv6
  encapsulation
   traffic-class propagate
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
### Others
  
  <h2> What is similar </h2>
As discussed in the Overview section, any type of service creation. show comamnds related to transport and service infra are common to all the NCS 500 and NCS 5500/5700 PIDs.

<h3> Summary </h2>
