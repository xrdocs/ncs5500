---
published: true
date: '2023-08-18 14:28 +0530'
title: 'Introducing NC57-48Q2D : Low Speed NCS 5700 Line Card'
author: Paban Sarma
excerpt: >-
  Introduction and deep dive to the NC57-48Q2D-(SE)-S, dense low speed line card
  in the cns 5700 modular portfolio. Discusses LC architecture and use cases in
  details. 
tags:
  - iosxr
position: hidden
---

|Paban Sarma, Technical Marketing Engineer, Cisco (pasarma@cisco.com)|
|Bala Murali Krishna, Technical Marketing Engineer, Cisco (bsanka@cisco.com)|


## Introduction
The Cisco NCS 5500 Series Modular Platforms offers industry-leading 100 GbE and 400 GbE port density to handle massive traffic growth. Latest trends in the Metro Architecture have driven the evolution of the product portfolio. Starting XR release 7.0.2 , the NC57 Line Cards are introduced in the platform to support higher speed and flexible scalablity. With IOS-XR 7.10.1 two new variants of line cards are introduced _viz._ NC57-48Q2D-S and NC57-48Q2D-SE-S, bringing highly scalable dense low speed line card to the family of NCS 5700 modular line card.

![xdocs.jpg]({{site.baseurl}}/images/xdocs.jpg)

_Figure 1: NC57-48Q2D-S/NC57-48Q2D-SE-S Front View_

## NC57 Series  Modular Line Cards for NCS 55xx Chassis
The NC57 Series Line Cards brought the support for 400G with enhanced scalability in the NCS 5500 Modular Chassis Family. The family of line card now offers speed range starting from 1G to 400G at unmatched scalability and functionality. The table below summarises the NC57 series line card family as of today.


_Table 1: List of NC57 Line Cards for NCS 55xx Modular Chassis_

| Line Card PID | Line Card Description | FCS Release |
|---------------|-----------------------|-------------|
| NC57-18DD-SE |18x400G or 30x200/100G Scale LC| IOS XR 7.0.2|
|NC57-24DD | 24x400G Base LC | IOS XR 7.0.2 |
| NC57-36H-SE |36x100G Scale Line Card |IOS XR 7.3.1|
| NC57-36H6D-S | 24x400G + 12 Flex Port (6x400GE or 12x200GE/100GE) base LC| IOS XR 7.4.1 |
|NC57-MOD-S| 2x400G + 8x50G + 2xMPA Base LC  |IOS XR 7.6.1| 
|<mark>NC57-48Q2D-S</mark>| <mark>32x25G+16x50G+2x400G Base LC </mark> |<mark>IOS XR 7.10.1</mark>|
|<mark>NC57-48Q2D-SE-S</mark>| <mark>32x25G+16x50G+2x400G Scale LC </mark> |<mark>IOS XR 7.10.1</mark>|

## NC57-48Q2D Video Introduction

_Placeholder  Video link_

## NC57-48Q2D Line Card Specification

The new NCS5700 line card variants are build to bring in dense low speed variant to the fold. The NC57-48Q2D-(SE)-S line card is comprised of multirate SFP28, SFP56 & QSFP56-DD ports. There are 32xSFP28 ports that can work at 1/10/25G , the 16xSFP56 ports can work at 10/25/50G and the 2xQDD port works 100/200/400G with Breakout support. 

![Vigor-LS-Ports.png]({{site.baseurl}}/images/Vigor-LS-Ports.png)
_Figure 2: NC57-48Q2D-(SE)-S Line Card Ports view

### Front Panel Ports and Speed
The NC57-48Q2D line card is built to serve as a dense low speed line and the first line card in NCS 5700 modular family to support native 1G. All the front panel ports are multirate in nature and the speed ranges from 1G to 400G. 

#### SFP28 Ports
As described _Figure 2_, The first 32 ports (P0 to P31) are of SFP28 Form factor and works at 1/10/25G speed. By default ports comes up as 25G, the speed can be changed with controller optics command per port. 

#### SFP56 Ports
The next 16 ports (P32 to P47) can support a speed upto 50G. These are true multirate port capabale of supporting 1/10/25/50G. 

`Note: 1G support on the SFP56 will be availble later release`

#### QSFP56DD Ports
Apart from the low speed SFP form factors, the line card variant is also equipped with 2 Nos. of QSFP56DD ports supporting upto 400G. The ports are back compitable with QSFP28 and support 40G/ 100G, 200G and breakout options.


### Internal Architecture

![vigor-ls-block.png]({{site.baseurl}}/images/vigor-ls-block.png)

The NC57-48Q2D-(SE)-S line card is built with single J2C ASIC and offers a massive throughput of 2.4 Tbps/1BPPS. The scaled variant of the line card, NC57-48Q2D-SE-S comes with OP2 external TCAM that assists in achieveing higher prefix and service scale. All the front panel ports of the line cards are connected to to network IFs of the NPU via PHY element which works as retimes and also enabled MACsec capability for all ports. 

The first set of 32 SFP28 ports are coming from 32 set of 25GE NIF on the ASIC via 2x PHY elements. The next set of 16xSFP56 Ports also connected via 2xPHY element , each taking 16x25GE NIF line and connecting 8xSFP56 ports at the faceplate. Thus each individual port works at 1/10/25/50G multirate. 

The QSFP56-DD ports takes out 16x50GE Serdes and each group of 8x50GE NIF is converted to one QDD 400G port at the faceplate. This is a true multirate port and natively works 40/100/200/400G speed. It can be also used in various breakout mode combination. 

### Port Speed Breakouts
Bu default, the NC57-48Q2D-(SE)-S line cards comes up as an 32x25G+16x50G+2x400G line cards. following snippet shows the default port speeds.

`
sho ip int br 
`

the speeds of each port can be set individully using the controller optics configuration. 

### MACsec & timing

## Scalability & Use Cases

## Summary & References

With the addition of the NC57-48Q2D line cards to the existing portfolio of 400G, 100G and modular line cards, the NCS 5500 modular systems can be deployed in a diverse set of use cases and unleash the full power of Jericho 2 native mode. More information on the line card and the NCS 5500/5700 product family can be found in the below links. 


- [NCS 5500/5700 @ Cisco.com](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html)
- [NCS5500/5700 Modular WhitePaper](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-convergence-sys-wp.html)
