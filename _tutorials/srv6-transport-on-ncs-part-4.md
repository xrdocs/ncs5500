---
published: true
date: '2022-11-14 15:25 +0530'
title: SRv6 Transport on NCS 5700
position: hidden
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Tejas Lad, Technical Marketing Engineer (telad@cisco.com)|

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
### Defining traffic-class encapsulations
### SRv6 Locator blocks
### Others
  
  <h2> What is similar </h2>
As discussed in the Overview section, any type of service creation. show comamnds related to transport and service infra are common to all the NCS 500 and NCS 5500/5700 PIDs.

<h3> Summary </h2>