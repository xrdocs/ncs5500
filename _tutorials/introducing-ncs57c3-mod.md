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

This article has been written and reviewed by (in alphabetical order):
- Akshaya Kumar Sankaran, TME
- Amit Kumar Dey, PM
- Nicolas Fevrier, TME
- Paban Sarma, TME
- Tejas Lad, TME
- Vincent Ng, TME

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

Coming SOON: video on port configuration, MACsec support and timing support.

## Understanding the naming logic

The name of the product is different depending on the licensing model used.  

With Flexible Consumption Model:  
- NCS-57C3-MOD-SYS is the "base" version
- NCS-57C3-MODS-SYS is the "scale" version (ie: equiped with External TCAM and half the numbers of 100G fixed ports)

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

![FCM.png]({{site.baseurl}}/images/FCM.png){: .align-center}

![BAU.png]({{site.baseurl}}/images/BAU.png){: .align-center}

## Product description

### Positioning

NCS57C3-MOD can be positioned in a very large variety of roles in the network due to its flexibility, compact form factor and high-level forwarding capacity. Non exhaustively:  
- 5G Mobile Backhaul (Class C, 1G to 400G)
- Core & Peering (MACSEC,100G, 400G, High Routing Scale)
- Enterprise & Residential Aggregation (MACSEC, 10G, 25G, 100G, Higher Aggregation Scale)
- Cloud Native Broadband Network Gateway (in roadmap)
- Routed Optical Networks (400G ZR/ZRP*, PLE*) *roadmap

### CCO Documentation

The product documentation is available here:  
- datasheet: [https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-57C3-fixed-chassis-ds.html](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-57C3-fixed-chassis-ds.html)
	- dimensions
    - power usage
    - PIDs
    - ports
    - supported standards
    - pretty much everything you need to know ;)
- fixed systems white paper: [https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-fixed-platform-architecture-white-paper.pdf](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-fixed-platform-architecture-white-paper.pdf)  
- installation guide: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/hardware-install/b-ncs5700-hardware-installation-guide-fixed-port/m-ncs-5700-router-overview.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/hardware-install/b-ncs5700-hardware-installation-guide-fixed-port/m-ncs-5700-router-overview.html)

### Software

The product is launched with IOS XR 7.4.1.

Note: the system is running XR 64bit and not "XR7". So it shares the same image than other modular and fixed chassis (except the NCS57B1 variants).
{: .notice--info}

In the download center, the search engine will not help with NCS 57C3 keywords:

![not-found.png]({{site.baseurl}}/images/not-found.png){: .align-center}

But you can pick any image NCS5500 image to upgrade your NCS57C3-MOD routers (except the one for NCS57B1, of course since it's XR7 one):

![xr-image-741.png]({{site.baseurl}}/images/xr-image-741.png){: .align-center}


### Hardware

We launch two versions of this new 3-RU modular system: base and scale.  
They only differ in two aspects: external TCAM to complete the J2C NPU and the number of QSFP ports.

|  | NCS-57C3-MOD-S / NCS-57C3-MOD-SYS | NCS-57C3-MOD-SE-S / NCS-57C3-MODS-SYS |
|:-----:|:-----:|:-----:|
| Fixed SFP 1G/10G/25G | 48 | 48 |
| Fixed QSFP 40G/100G | 8 | 4 |
| MPA bays | 2x 800G + 1x 400G | 2x 800G + 1x 400G |
| Dual RP | Yes | Yes |
| Forwarding ASIC | J2C | J2C |
| eTCAM | No | OP2 |
| Total interfaces | 4Tbps | 3.6Tbps |
| NPU Forwarding | 2.4Tbps | 2.4 Tbps |

Fixed ports (interfaces), Modular Port Adaptors (MPA) slots, Route Processors (RPs) and Power Supply Units (PSUs) are reachable from the front:

![dimensions-base.png]({{site.baseurl}}/images/dimensions-base.png){: .align-center}

![dimensions-SE-.png]({{site.baseurl}}/images/dimensions-SE-.png){: .align-center}

Only Fan trays are reachable from the back:

![back-flat.jpg]({{site.baseurl}}/images/back-flat.jpg){: .align-center}

### LEDs and Displays

Coming very soon.

### Route Processors

The NCS57C3-MOD is the first of the "fixed platforms" portfolio to offer control plane redundancy via the presence of these two NC57-MOD-RP2-E inserted in the front of the router (top right two slots).  

![RP2-side.jpg]({{site.baseurl}}/images/RP2-side.jpg){: .align-center}

Note: the system can operate in nominal manner with only one RP. The dual RP is optional.  
As mentioned above, it offers control plane redundancy and not forwarding plane redundancy. It means the protocols and processes will be checkpointed between the two route processors, in the same way it's done on the chassis 5504/5508/5516, enabling the NSR/NSF/GR features between protocols. But at the difference of the chassis, we don't have multiple fabric cards (no fabric card at all).  
The RP doesn't contain the Jericho2C NPU, it's located in an "internal line card".

You'll find details on the CPU type, memories, etc in the datasheet linked above.

Notes:  
- the NC57-MOD-RP2-E is specific to the NCS57C3 and can't be used in the modular chassis (or vice versa), it's a totally different form factor
- we don't have plans to implement ISSU on this system. The RP doesn't contain the forwarding device, so at best we should be able to speed up the reload process but not reach true ISSU capabilities.

### Power supply

In the bottom left side of the front, we can insert two PSUs. Two flavors exist: 1600W AC or 1600W DC. Both AC or DC options offer 1+1 redundancy.

![AC1600-.jpg]({{site.baseurl}}/images/AC1600-.jpg){: .align-center}

![DC1600-.jpg]({{site.baseurl}}/images/DC1600-.jpg){: .align-center}

Mixed (AC+DC) is possible but only tested for short period of time (like during a live migration).

### Fan trays

The cooling of the chassis is guaranteed by a system of 6 fan trays inserted in the back of the box. It's a front to back design with two types of FTs:

NC57-C3-FAN1-FW (40mm):  
![NC57-C3-FAN1-FW-side-.jpg]({{site.baseurl}}/images/NC57-C3-FAN1-FW-side-.jpg){: .align-center}

NC57-C3-FAN2-FW (60mm):  
![NC57-C3-FAN2-FW-side-.jpg]({{site.baseurl}}/images/NC57-C3-FAN2-FW-side-.jpg){: .align-center}

From a back perspective, the fan slots are numbered from left to right, 0 to 5.

![Fans-Numbering.png]({{site.baseurl}}/images/Fans-Numbering.png){: .align-center}

|  | NC57-C3-FAN1-FW | NC57-C3-FAN2-FW |
|:-----:|:-----:|:-----:|
| Size | 40mm | 60mm |
| Position/Slot | 2/3/4/5 | 0/1 |

In normal conditions (all 6 fan trays active), the system can operate at:  
- 50C (at 1800m)
- 45C when using QSFP-DD MPA with low powered optics 

With a single fan failure (whether it is NC57-C3-FAN1-FW or NC57-C3-FAN2-FW), the system can operate at 40C.

### Modular Port Adaptors

![MPA-numbered.png]({{site.baseurl}}/images/MPA-numbered.png){: .align-center}

NCS57C3-MOD offers three MPA bays:
- 2 for 800Gbps MPA (or 400G): slot 1/2
- 1 for 400Gbps MPA only: slot 0

All slots support existing MPAs:
- NC55-MPA-2TH-S: NCS 5500 2X200G CFP2 MPA
- NC55-MPA-1TH2H-S: NCS 5500 1X200G CFP2 + 2X100G QSFP28 MPA
- NC55-MPA-12T-S: NCS 5500 12X10G MPA
- NC55-MPA-4H-S: NCS 5500 4X100G QSFP28 MPA

And slots 1 and 2 support new generation MPAs at 800G:

- NC57-MPA-2D4H-S (New): NCS 5700 4X QSFP-DD MPA

More details to come very soon with the second video 

## Ports identification

![NCS57C3-MOD-S.png]({{site.baseurl}}/images/NCS57C3-MOD-S.png){: .align-center}

![NCS57C3-MOD-SE-S.png]({{site.baseurl}}/images/NCS57C3-MOD-SE-S.png){: .align-center}



## Forwarding ASIC (NPU)

Presentation J2C  
Differences with J2  

|  | Jericho2 | Jericho2C |
|:-----:|:-----:|:-----:|
| xxx | x | x |


## Block Diagrams

![block-diagram.png]({{site.baseurl}}/images/block-diagram.png){: .align-center}

Interesting to note the NCS57C3-MOD-SYS and NCS57C3-MODS-SYS are SoC (system on the chip). That means all the ports are directly connected to a single forwarding ASIC.



![EPC-EOBC.png]({{site.baseurl}}/images/EPC-EOBC.png){: .align-center}


## MACsec

Will be completed with next videos

## Timing

Will be completed with next videos


