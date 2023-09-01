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

### Port Speeds &  Breakouts Options

Bu default, the NC57-48Q2D-(SE)-S line cards comes up as an 32x25G+16x50G+2x400G line cards. following snippet shows the default port speeds. 
   
   <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int br | in E0/5
Mon Aug 29 19:57:58.648 UTC
TwentyFiveGigE0/5/0/0          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/1          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/2          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/3          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/4          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/5          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/6          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/7          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/8          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/9          unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/10         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/11         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/12         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/13         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/14         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/15         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/16         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/17         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/18         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/19         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/20         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/21         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/22         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/23         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/24         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/25         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/26         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/27         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/28         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/29         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/30         unassigned      Shutdown        Down     default 
TwentyFiveGigE0/5/0/31         unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/32              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/33              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/34              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/35              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/36              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/37              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/38              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/39              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/40              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/41              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/42              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/43              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/44              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/45              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/46              unassigned      Shutdown        Down     default 
FiftyGigE0/5/0/47              unassigned      Shutdown        Down     default 
FourHundredGigE0/5/0/48        unassigned      Shutdown        Down     default 
FourHundredGigE0/5/0/49        unassigned      Shutdown        Down     default 
      </code>
     </pre>
    </div>

The speeds of each port can be set individully using the controller optics configuration. Different supported breakout modes are also configured under the controller optics configuration for the QDD ports.

_Snippet showing 100g speed on QDD port_
   <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
      controller Optics0/5/0/48
		 speed 100g
		!
       
       RP/0/RP0/CPU0:NCS5508-II9-43#sho ipv4 int br | i 0/5/0/48
	   Mon Aug 29 20:08:46.900 UTC
	   <mark>HundredGigE0/5/0/48            unassigned      Down            Down    default</mark> 
      </code>
     </pre>
    </div>
    
_Snippet showing 4x100G  breakout on QDD port_  
    <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
controller Optics0/5/0/48
 breakout 4x100

RP/0/RP0/CPU0:NCS5508-II9-43#sho ipv4 int br | i 0/5/0/48
Mon Aug 29 20:07:42.253 UTC
<mark> HundredGigE0/5/0/48/0          unassigned      Down            Down     default </mark>
<mark>HundredGigE0/5/0/48/1          unassigned      Down            Down     default </mark>
<mark>HundredGigE0/5/0/48/2          unassigned      Down            Down     default </mark>
<mark>HundredGigE0/5/0/48/3          unassigned      Down            Down     default </mark>
     </code>
     </pre>
    </div>
 
 _Snippet showing 4x25G  breakout on QDD port_
 <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
     controller Optics0/5/0/48
      speed 100g
      breakout 4x25

RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int br | in 0/5/0/48
Tue Aug 30 17:36:51.439 UTC
<mark>TwentyFiveGigE0/5/0/48/0       unassigned      Down            Down     default </mark>
<mark>TwentyFiveGigE0/5/0/48/1       unassigned      Down            Down     default </mark>
<mark>TwentyFiveGigE0/5/0/48/2       unassigned      Down            Down     default </mark>
<mark>TwentyFiveGigE0/5/0/48/3       unassigned      Down            Down     default </mark>

      </code>
     </pre>
    </div>
    
_Snippet showing  changing speed for SFP56 ports_

 <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
controller Optics0/5/0/32
 speed 25g
!
controller Optics0/5/0/33
 speed 10g
!

RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int brief | in 0/5/0/32
Tue Aug 30 18:25:21.057 UTC
<mark>TwentyFiveGigE0/5/0/32         unassigned      Down            Down     default </mark>

RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int brief | in 0/5/0/33
Tue Aug 30 18:25:22.564 UTC
<mark>TenGigE0/5/0/33                unassigned      Down            Down     default </mark>
 </code>
     </pre>
    </div>
    

On the NC5748Q-2D series line cards, port speeds are reseted based on inserted optics as well. for example, when 1G SFP is inserted on the SFP28 ports or a 10G or 25G optics is inserted in the SFP 56 ports, the corresponding  ports will be set to a GigabitE or TenGE or TwentyFiveGE port respectively. 

 <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
 

RP/0/RP0/CPU0:NCS5508-II9-43#show  run controller optics 0/5/0/34
Tue Aug 30 21:22:51.676 UTC
% No such configuration item(s)
RP/0/RP0/CPU0:NCS5508-II9-43#show  inventory  | in 25G   
Tue Aug 30 21:23:02.160 UTC
NAME: "0/5", DESCR: "NCS 5700 32x1/10/25G + 16x1/10/25/50G + 2x400G Line Card BASE"
<mark>NAME: "TwentyFiveGigE0/5/0/34", DESCR: "Cisco SFP28 25G SR Pluggable Optics Module"</mark>
PID: SFP-25G-SR-S      , VID: V01, SN: INL2453AF2M
RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int br | in 0/5/0/34
Tue Aug 30 21:23:20.934 UTC
<mark>TwentyFiveGigE0/5/0/34</mark>         unassigned      Shutdown        Down     default 


RP/0/RP0/CPU0:NCS5508-II9-43#show  inventory  | in 1G 
Tue Aug 30 21:25:08.065 UTC
<mark>NAME: "GigabitEthernet0/5/0/20", DESCR: "Cisco SFP 1G 1000BASE-SX Pluggable Optics Module </mark>
RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int br | in 0/5/0/20
Tue Aug 30 21:25:20.803 UTC
<mark>GigabitEthernet0/5/0/20        unassigned      Shutdown        Down     default </mark>
RP/0/RP0/CPU0:NCS5508-II9-43#show  run controller optics 0/5/0/20
Tue Aug 30 21:25:29.688 UTC
% No such configuration item(s)



RP/0/RP0/CPU0:NCS5508-II9-43#show  run controller optics 0/5/0/48
Tue Aug 30 21:28:35.462 UTC
% No such configuration item(s)
RP/0/RP0/CPU0:NCS5508-II9-43#show  ipv4 int br | in 0/5/0/48
Tue Aug 30 21:28:46.336 UTC
<mark>FortyGigE0/5/0/48</mark>              unassigned      Shutdown        Down     default 
RP/0/RP0/CPU0:NCS5508-II9-43#show  inventory | in Forty
Tue Aug 30 21:28:52.485 UTC
<mark>NAME: "FortyGigE0/5/0/48", DESCR: "Cisco QSFP+ 40G SR4 Pluggable Optics Module"</mark>
RP/0/RP0/CPU0:NCS5508-II9-43#
 </code>
     </pre>
    </div>

### MACsec & Timing
The NC57-48Q2D series line card supports MACsec encryption on all the ports. This is achieved by the use of the PHY element. The following table summerize, MACsec support at different port speed for the type of ports.

| SFP 28 ports 0-31 | SPF56 ports 32-47 | QDD56 Ports 48,49|
|-------------------|-------------------|------------------|
|1G, 10G,25G | 10G, 25G, 50G | 4x10G/40G/4x25G/100G/2x100G/4x100G/400G|

`Note: 1G MAcsec support on the SFP56 ports (32-47) are not available in the initail release and`

This line card variants support industry standard class C timing on all the ports when the modular chasis is equiped with NC55-RP2-E card. 


## Scalability & Use Cases

## Summary & References

With the addition of the NC57-48Q2D line cards to the existing portfolio of 400G, 100G and modular line cards, the NCS 5500 modular systems can be deployed in a diverse set of use cases and unleash the full power of Jericho 2 native mode. More information on the line card and the NCS 5500/5700 product family can be found in the below links. 


- [NCS 5500/5700 @ Cisco.com](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html)
- [NCS5500/5700 Modular WhitePaper](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-convergence-sys-wp.html)
