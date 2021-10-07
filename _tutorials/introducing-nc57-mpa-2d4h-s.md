---
published: true
date: '2021-10-07 14:55 +0200'
title: 'Introducing the new QSFP-DD MPA: NC57-MPA-2D4H-S'
author: Nicolas Fevrier
excerpt: Introducing the new QSFP-DD modular port adaptor
position: hidden
---
{% include toc icon="table" title="New QSFP-DD MPA: NC57-MPA-2D4H-S" %} 

This article has been written and reviewed by (in alphabetical order):
- Akshaya Kumar Sankaran, TME
- Amit Kumar Dey, PM
- Nicolas Fevrier, TME
- Paban Sarma, TME
- Tejas Lad, TME
- Vincent Ng, TME

## Introduction

With IOS XR 7.4.1, we introduced [a lot of new features](https://xrdocs.io/ncs5500/tutorials/iosxr-741-innovations/) and new routers like [the Cisco NCS57C3-MOD](https://xrdocs.io/ncs5500/tutorials/introducing-ncs57c3-mod/), but that's not the only new hardware launched with this new release. We are very pleased to announce the availability of a new generation of Modular Port Adapter, with QSFP-DD support: the NC57-MPA-2D4H-S.

This new MPA will be supported in the different slots of the NCS57C3-MOD routers, but also in the very popular NCS55A2-MOD series and the modular line cards NC55-MOD-A-S and NC55-MOD-A-SE-S.  
So it will be supported in the existing 400Gbps MPA slots, protecting your investment, but also in the new 800Gbps slots available in the NCS57C3-MOD routers, unleashing its fully capability.

## Video

In this one, Vincent presents both the NCS57C3-MOD specificities and the NC57-MPA-2D4H-S module (in the second part starting at 3:15 [https://www.youtube.com/watch?v=MV2hNv4xn6Q&t=196s](https://www.youtube.com/watch?v=MV2hNv4xn6Q&t=196s)) 

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/MV2hNv4xn6Q" frameborder="0" allowfullscreen></iframe>{: .align-center}


## CCO documentation

- datasheet: [https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-5700-series-mpa-ds.html](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-5700-series-mpa-ds.html)
- TMG matrix for supported optics: [https://tmgmatrix.cisco.com/?npid=4661](https://tmgmatrix.cisco.com/?npid=4661)

## Product Description

The form factor is comparable to the 400Gbps version available today (NC55-MPA-2TH-S, NC55-MPA-1TH2H-S, NC55-MPA-12T-S and NC55-MPA-4H-S).

![Side-view-blue.png]({{site.baseurl}}/images/Side-view-blue.png){: .align-center}

This MPA is 800Gbps capable and can accomodate a variety of port mixing combinations.  
The 4 ports are QSFP-DD cages, making it ready for ZR/ZRP optics (100G and 400G). It's the perfect fit for the Routed Optical Network architectures, [you can find more details on this design here](https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking.html).

This MPA is useable in multiple NCS5500 and NCS5700 products and line cards, we will detail them now, one by one.

## In NCS57C3-MOD Routers

Please check [this artcle dedicated to the NCS57C3-MOD routers first](https://xrdocs.io/ncs5500/tutorials/introducing-ncs57c3-mod/).

This fixed/modular router is the first of its kind in the portfolio, and its flexibility comes primarily from the three MPA slots. One at 400Gbps (slot 1) and two at 800Gbps (slot 2 and 3).

![MPA-numbered.png]({{site.baseurl}}/images/MPA-numbered.png){: .align-center}

The module presents 4 QDD ports in the front, connected to a PHY chipset, capable of different behaviors (reverse gear box, forward gear box, retimer, ...) and handling features like timing and MACsec encryption.

![MPA-block-diagram.png]({{site.baseurl}}/images/MPA-block-diagram.png){: .align-center}

Depending on the slot we will insert it into, the NC57-MPA-2D4H-S will offer different capabilities and support different port combination. 

### In 800Gbps Mode

When inserted in slot 2 and 3, the MPA offers 800Gbps of throughput, 400G for each pair of ports (0-1 and 2-3).

![400+400.png]({{site.baseurl}}/images/400+400.png){: .align-center}

| Port 0/2 | Port 1/3 |
|:-----:|:-----:|
| 400G or 4x100G | Empty |
| QDD-2x100G | QDD-2x100G |
| 100G or 40G or 4x25G or 4x10G | 100G or 40G or 4x25G or 4x10G |
| QDD-2x100G | 100G or 4x25G |
| 100G or 4x25G | QDD-2x100G |

100G/400G-ZR/ZR+ Transponder and Nx100G Muxponder modes are in the roadmap. Contact your Cisco representative if you need more details.

### In 400Gbps Mode

When inserted in slot 1, the MPA offers 400Gbps of throughput total.

![1x400.png]({{site.baseurl}}/images/1x400.png){: .align-center}

We can't offer a single 400GE port in this mode, only 4x100G in port 0 or multiple ports of diverse speed as shown in the chart:

| Port 0 | Port 1 | Port 2 | Port 3 |
|:-----:|:-----:|:-----:|:-----:|
| 4x100G breakout | Disabled | Disabled | Disabled |
| QDD-2x100G | Disabled | QDD-2x100G | Disabled |
| 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G |
| QDD2x100G | Disabled | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G |
| 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | QDD2x100G | Disabled |

100G/400G-ZR/ZR+ Nx100G Muxponder modes are in the roadmap. Contact your Cisco representative if you need more details.

### MACsec Support

All speeds 400G, 100G, 40G supported such as MACsec over breakout (4x100G, 4x25G and 4x10G).

### Timing Support

All speeds 400G, 100G, 40G support SyncE and PTP:  
- G.8275.1 Telecom Profile supported with Class C Performance
- G.8275.2 Telecom Profile also supported

## In NCS55A2-MOD Routers

The two MPA slots being 400G, it's the same port capability than 400Gbps mode described above. We don't support 1x400G Grey but will support 4x100G breakout. For 400G-ZR/ZR+ Coherent in the future plan, we will support Nx100G Muxponder modes. Current optics speeds and breakout supported are as follows:

| Port 0 | Port 1 | Port 2 | Port 3 |
|:-----:|:-----:|:-----:|:-----:|
| 4x100G breakout | Disabled | Disabled | Disabled |
| QDD-2x100G | Disabled | QDD-2x100G | Disabled |
| 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G |
| QDD2x100G | Disabled | 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G |
| 100G/40G/4x25G/4x10G | 100G/40G/4x25G/4x10G | QDD2x100G | Disabled |

Timing and MACsec support are not available at 7.4.1 and are currently tracked in the roadmap, contact your Cisco representative for more details.

### Use case?

Most obvious question here will be: why not using the good ol' NC55-MPA-4H-S since you can't afford more than 4 ports 100G here?

The main use case for using an NC57-MPA-2D4H-S in NCS55A2-MOD will be the Route Optical (RON) architectures. Indeed, for ZR and ZRP optics support at 100G, you need a QSFP-DD cage. So, using coherent ZR/ZR+ optics in 100G, 4x100G or 2x100G will require the new MPA.

Note: support of ZR/ZRP is in the roadmap and currently not available in IOS XR 7.4.1 at FCS.

## In NC55-MOD-A Line Cards

Currently tracked in the roadmap, contact your Cisco representative for more details.
