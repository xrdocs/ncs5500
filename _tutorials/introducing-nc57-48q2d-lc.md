---
published: false
date: '2023-05-18 14:28 +0530'
title: 'Introducing NC57-48Q2D : Low Speed NCS 5700 Line Card'
author: Paban Sarma
excerpt: >-
  Introduction and deep dive to the NC57-48Q2D-(SE)-S, dense low speed line card
  in the cns 5700 modular portfolio. Discusses LC architecture and use cases in
  details. 
tags:
  - iosxr
---
## Introduction
The Cisco NCS 5500 Series Modular Platforms offers industry-leading 100 GbE and 400 GbE port density to handle massive traffic growth. Latest trends in the Metro Architecture have driven the evolution of the product portfolio. Starting XR release 7.0.2 , the NC57 Line Cards are introduced in the platform to support higher speed and flexible scalablity. With IOS-XR 7.10.1 two new variants of line cards are introduced _viz._ NC57-48Q2D-S and NC57-48Q2D-SE-S, bringing highly scalable dense low speed line card to the family of NCS 5700 modular line card.

_Placeholder for Image_


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

_Placeholder for picture with Port combination view_

### Internal Architecture
The NC57-48Q2D-(SE)-S line card is built with single J2C ASIC and offers a massive throughput of 2.4 Tbps/1BPPS. The scaled variant of the line card, NC57-48Q2D-SE-S comes with OP2 external TCAM that assists in achieveing higher prefix and service scale. 

### Front Panel Ports and Speed

### MACsec & timing

## Scalability & Use Cases

## Summary & References

With the addition of the NC57-48Q2D line cards to the existing portfolio of 400G, 100G and modular line cards, the NCS 5500 modular systems can be deployed in a diverse set of use cases and unleash the full power of Jericho 2 native mode. More information on the line card and the NCS 5500/5700 product family can be found in the below links. 


- [NCS 5500/5700 @ Cisco.com](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html)
- [NCS5500/5700 Modular WhitePaper](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-convergence-sys-wp.html)
