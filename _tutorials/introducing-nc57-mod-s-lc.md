---
published: false
date: '2023-05-08 09:36 +0530'
title: 'Introducing NC57-MOD-S: NCS 5700 Modular Line Card'
author: Paban Sarma
position: hidden
excerpt: >-
  Introduction to Cisco NC57-MOD-S Line card. A modular Line card for NCS 5500
  Modular Chassis. Detailed LC architecture, Capabilities and Use Cases.
---
## Introduction

The Cisco NCS 5500 Series Modular Platforms offer industry-leading 100 GbE and 400 GbE port density to handle massive traffic growth. Latest Trends in the Metro Architecture has driven the evoltion of The product portfolio. Staring XR release 7.0.2 , the NC57 Line Cards are introduced in the platform to support higher speed and flerxible scalablity. With IOS XR 7.6.1, we have introduced the Modular Flavour line card NC57-MOD-S, which support a great flexibility in terms of supported speed and form factors of optics. 

![NC57-MOD-S.png]({{site.baseurl}}/images/NC57-MOD-S.png)
_Figure 1: The New Cisco NC57-MOD-S Modular Line Card_


## NC57 Series  Modular Line Cards for NCS 55xx Chassis
The NC57 Series Line Cards brought the support for 400G with enhanced scalability in the NCS 5500 Modlar Chassis Family. The table summeriese the NC57 series line card family as of today. 


*_Table 1: List of NC57 Line Cards for NCS 55xx Modular Chassis_*

| Line Card PID | Line Card Description | FCS Release |
|---------------|-----------------------|-------------|
| NC57-18DD-SE |18x400G or 30x200/100G Scale LC| IOS XR 7.0.2|
|NC57-24DD | 24x400G Base LC | IOS XR 7.0.2 |
| NC57-36H6D-S | 24x400G + 12 Flex Port (6x400GE or 12x200GE/100GE) base LC| |
| NC57-36H-SE |36x100G Scale Line Card ||
|<mark>NC57-MOD-S</mark>| <mark>2x400G + 8x50G + 2xMPA Base LC </mark> |<mark>IOS XR 7.6.1</mark>| 


## NC57-MOD-S Line Card Specification


### Line Card Architecture

NC57-MOD-S is a modular line card available only in Base version. It is based on a single Jericho2 ASIC. It is the successor of the NC5500 modular Line card (NC55-MOD-S). It offers a mix of fixed port and two slots for modular port adapters (MPAs). The Fixed ports includes 2xQSFP56-DD 100/200/400G ports and 8xSFP56 offering 10/25/50G. The MPA slots have a maximum throughput of 800G each. 

`Note: 1G port speed are not supported on the NC57-MOD line card on fixed or MPA based SFP ports`

![NC57-MOD-S-FV.png]({{site.baseurl}}/images/NC57-MOD-S-FV.png)
_Figure 2: NC57-MOD-S Modular Line Card Fixed Ports & MPA slots_



![NC57-MOD-S-FV.png]({{site.baseurl}}/images/NC57-MOD-S-Arch.png)
_Figure 3: NC57-MOD-S Modular Line Card Archtecture_

A single J2 ASIC is the primary building block of the Line card. It has another LC CPU complex and DRAM for deep buffering. The NIF interfaces are connected to the fixed front panel ports via a PHY that enables MACsec capability in those ports. the two MPA slots are directly connected via a group of NIFs allowing a maximum throughput of 800Gbps per MPA slots. Most of the MPAs contain PHY internally enabling support  MACsec encryption.

### Dimensions , Weights etc.

## Modular Port Adapters
As Mentioned earlier the two MPA slots on the NC57-MOD-S line card have a bandwidth upto 800 Gbps/slot. It supports the new generation of MPAs (Upto 800G/MPA) supporting form factors like QSFP56-DD, 400G CFP2DCO, SFP56 etc. The line card is also back compitable with the previous generation of MPAs (Upto 400G/MPA). The below table lists out all the MPA modules supported on the line card.

*_Table 2: List Supported MPAs on NC57-MOD Line Card_*

| MPA PID | MPA Port Configuration | MACSec Support|
|---------|------------------------|---------------|
|NC57-MPA-2D4H| 2x400G QDD or 4x100/200G QDD| Yes|
|NC57-MPA-1FH1D| 1x400G CFP2 + 1x400G QDD | Yes|
|NC57-MPA-12L | 12xSFP56 (10/25/50G) | Yes |
|NC55-MPA-4H-S| 4xQSFP28 (40/100G)| Yes|
| NC55-MPA-12T-S | 12xSFP+ (10G)|Yes|
|NC55-MPA-2TH-S | 2xCFP2DCO (200G)| Yes |
|NC55-MPA-1TH2H-S | 1xCFP2DCO (100G) + 2xQSFP28 (40/100G) | Yes|


## MACsec & Timing

MACsec is supported on all the fixed port on the NC57-MOD line card. As shown in figure above, MACsec support is achived via the MACsec capable PHY in the system. for the MPA ports, MACsec support on all the MACsec capable MPAs. This is listed in the _table 2_ above.

NC57-MOD line card supports all the timing functionality supported by NCS 5500 modular platforms. The line card is capable of supporting PTP Class-C timing accuracy, When the Chassis is equiped with NC55-RP2-E as the route processor.


## Scalability & Use Cases

As it is built on the J2 generation of BCM ASIC, NC57-MOD LC also supports the modular Database profiles (MDB) for  scale. We already covered the [MDB profiles](https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/) in our previous article. Since the line card comes only in base veriosn,  it can be configured with both L2MAX and L3MAX MDB profiles when operated in J2 native Mode. 

NC57-MOD-S Line card can be inserted in all the three variants of NCS 5500 modular chassis, viz. 5504, 5508 and 5516. These modular chassis equiped with this line card can be positioned in various roles such as Metro Aggregation , Core and Peering etc. Also, the support of ZR/ZR+ and 400G CFP2DCO optics via the MPA-1FH-1D make this line card an important piece in Cisco's [Routed Optical Networking](https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking/index.html) Architecture.


## Summary & References
The Cisco NCS 5700 modular line card NC57-MOD-S is a great choice for a versatile, high-performance networking solution. 


- [NCS 5500/5700 @ Cisco.com](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html)
- [NCS 5500/5700 XRdocs](https://xrdocs.io/ncs5500/)
- [NCS5500/5700 Modular WhitePaper](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-convergence-sys-wp.html)
