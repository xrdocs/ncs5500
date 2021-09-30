---
published: true
date: '2021-09-29 17:13 +0200'
title: Introducing NCS57C3-MOD Routers
author: Nicolas Fevrire
excerpt: >-
  Very happy to announce the new member of the NCS5500 fixed routers: the
  NCS57C3-MOD
position: hidden
---
{% include toc icon="table" title="Cisco NCS 57C3 MOD Routers" %} 

## Introduction

With IOS XR 7.4.1, we introduced multiple software features ([https://xrdocs.io/ncs5500/tutorials/iosxr-731-innovations/](https://xrdocs.io/ncs5500/tutorials/iosxr-731-innovations/)) but new hardware are also launched with this release. We are very happy to introduce a new member to the NCS5500 family, the NCS57C3-MOD series.

These two new routers are the NCS-57C3-MOD-SYS and NCS-57C3-MODS-SYS, that can be considered the successors of the NCS55A2-MOD since they are built following the same philosophy:  
- compact form-factor (less than 300mm deep and 3RU here)
-offering the highest level of flexibility with both fixed SFP and QSFP ports (1G, 10G, 25G, 40G, 100G) 
- modular port adaptors
but also new goodies are specific to this NCS57C3-MOD:  
- a 2.4Tbps of forwarding capability (compared to the 900G on NCS55A2-MOD)
- dual RP for control plane redundancy
- 3x MPA (two at 800G and one at 400G)

They can be used in multiple places in the network: aggregation, pre-agg, 5G (class-C capable), internet peering, core, enterprise...  

## Videos

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/ARKLok7dj-w" frameborder="0" allowfullscreen></iframe>{: .align-center}
.


## Understanding the naming logic

The name of the product is different depending on the licensing model used.  

With Flexible Consumption Model:  
- NCS-57C3-MOD-SYS is the "base" version
- NCS-57C3-MODS-SYS is the "scale" version (ie: equipped with External TCAM and half the numbers of 100G fixed ports)

With Perpetual / Business As Usual model:  
- NCS-57C3-MOD-S is the "base" version
- NCS-57C3-MOD-SE-S is the "scale" version (eTCAM)

Note that a "show platform" will display the FCM naming:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:ios#show plat
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/CPU0          <mark>NCS-57C3-MODS-SYS</mark>          IOS XR RUN        NSHUT
0/0/NPU0          Slice                      UP                
0/RP0/CPU0        NC57-MOD-RP2-E(Active)     IOS XR RUN        NSHUT
0/RP1/CPU0        NC57-MOD-RP2-E(Standby)    IOS XR RUN        NSHUT
0/FT0             NC57-C3-FAN2-FW            OPERATIONAL       NSHUT
0/FT1             NC57-C3-FAN2-FW            OPERATIONAL       NSHUT
0/FT2             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT3             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT4             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT5             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/PM0             NC57-1600W-ACFW            OPERATIONAL       NSHUT
RP/0/RP0/CPU0:ios#</code>
</pre>
</div>

Or in admin mode:

<div class="highlighter-rouge">
<pre class="highlight">
<code>sysadmin-vm:0_RP0# show controller card-mgr inventory summary

Card Manager Inventory Summary :
                               BP                 HW
Location Card Type             ID  Serial Number  Ver    Card State
------------------------------------------------------------------------------
0/0      <mark>NCS-57C3-MODS-SYS</mark>     1   FOCxxxxxxxx    0.1    CARD_READY
0/RP0    NC57-MOD-RP2-E (Master) 27  FOCxxxxxxxx    1.0    CARD_READY
0/RP1    NC57-MOD-RP2-E (Slave) 28  FOCxxxxxxxx    1.0    CARD_READY
</code>
</pre>
</div>




