---
published: true
date: '2021-10-07 14:55 +0200'
title: 'Introducing the new QSFP-DD MPA: NC57-MPA-2D4H-S'
author: Nicolas Fevrier
excerpt: Introducing the new QSFP-DD modular port adaptor
position: top
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

Vincent Ng presents the NC57-MPA-2D4H-S module:

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/6Ksv8oqhBk0" frameborder="0" allowfullscreen></iframe>{: .align-center}

## CCO documentation

- Datasheet:  
[https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-5700-series-mpa-ds.html](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-5700-series-mpa-ds.html)
- TMG matrix for supported optics:  
[https://tmgmatrix.cisco.com/?npid=4661](https://tmgmatrix.cisco.com/?npid=4661)
- Visio stencils:  
[https://www.cisco.com/c/en/us/products/visio-stencil-listing.html](https://www.cisco.com/c/en/us/products/visio-stencil-listing.html)


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

### Configuring Interface Speed 400G on the MPA

As discussed in the above section we can use the MPA is either slot 1,2,3. It will operate in either 400G or 800G mode. Now we will see the default mode of the ports when the MPA boots up and how to configure a port for 400G speed.

We have a router with the MPA in slot 1 and slot 2. We will concentrate on slot 2 for this example

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#show platform
Thu Mar  3 11:21:37.216 UTC
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/1             NC57-MPA-2D4H-S            OK                
<mark>0/0/2             NC57-MPA-2D4H-S            OK </mark>               
0/0/3             NC57-MPA-12L-S             OK                
0/0/CPU0          NCS-57C3-MODS-SYS          IOS XR RUN        NSHUT
0/0/NPU0          Slice                      UP                
0/RP0/CPU0        NC57-MOD-RP2-E(Active)     IOS XR RUN        NSHUT
0/FT0             NC57-C3-FAN2-FW            OPERATIONAL       NSHUT
0/FT1             NC57-C3-FAN2-FW            OPERATIONAL       NSHUT
0/FT2             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT3             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT4             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/FT5             NC57-C3-FAN1-FW            OPERATIONAL       NSHUT
0/PM0             NC57-1600W-ACFW            OPERATIONAL       NSHUT
0/PM1             NC57-1600W-ACFW            FAILED            NSHUT
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#
</code>
</pre>
</div>

We have inserted 400G optics in slot 0 and slot 2

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#show inventory 
NAME: "0/0", DESCR: "NCS 5700 Eyrie Line Card"
PID: NCS-57C3-MODS-SYS , VID: V01, SN: FOC25296E5Z

NAME: "0/0/1", DESCR: "2X400G or 4X200/100G QSFP-DD MPA"
PID: NC57-MPA-2D4H-S   , VID: V01, SN: FOC252925T7

NAME: "0/0/2", DESCR: "2X400G or 4X200/100G QSFP-DD MPA"
PID: NC57-MPA-2D4H-S   , VID: V01, SN: FOC252925T8

<mark>NAME: "HundredGigE0/0/2/0", DESCR: "Cisco QSFPDD 400G AOC Pluggable Optics Module"
PID: QDD-400-AOC3M     , VID: V01 , SN: INL2528A9ZV-B</mark>

<mark>NAME: "HundredGigE0/0/2/2", DESCR: "Cisco QSFPDD 400G AOC Pluggable Optics Module"
PID: QDD-400-AOC3M     , VID: V01 , SN: INL2528A9XN-A</mark>
</code>
</pre>
</div>


We can see that when the MPA boots up the interface comes up as 100G by default. It does not automatically take into consideration the inserted optics.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#show ipv4 interface brief | in 0/0/2
<mark>HundredGigE0/0/2/0             unassigned      Down            Down     default 
HundredGigE0/0/2/1             unassigned      Down            Down     default 
HundredGigE0/0/2/2             unassigned      Down            Down     default 
HundredGigE0/0/2/3             unassigned      Down            Down     default </mark>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#
</code>
</pre>
</div>

For configuring the port as 400G, we need to execute the below commands

```
hw-module port-range 0 1 instance 2 location 0/0/CPU0 mode 400
hw-module port-range 2 3 instance 2 location 0/0/CPU0 mode 400
```

| CLI portion | Meaning                                |
|-------------|----------------------------------------|
| port-range  | either 0-1 or 2-3                      |
| instance    | card instance of MPA's                 |
| location    | fully qualified location specification |
| mode        | port mode                              |

After configuring the above profile the interface comes up in 400G mode.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#show ipv4 interface brief  | in FourHundredGigE0/0/2
<mark>FourHundredGigE0/0/2/0         unassigned      Up              Up       default 
FourHundredGigE0/0/2/2         unassigned      Up              Up       default </mark>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#
</code>
</pre>
</div>

### Configuring one breakout option on the MPA

As seen in the above example, if we want to configure a port for a breakout option, please use the below CLI

```
hw-module port-range 0 1 instance 2 location 0/0/CPU0 mode 4x100
```
In this example, the 400G port in slot 0 is used in 4x100G mode. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#show ip int brief | in 0/0/2/0/
<mark>HundredGigE0/0/2/0/0           unassigned      Down            Down     default 
HundredGigE0/0/2/0/1           unassigned      Down            Down     default 
HundredGigE0/0/2/0/2           unassigned      Down            Down     default 
HundredGigE0/0/2/0/3           unassigned      Down            Down     default</mark> 
RP/0/RP0/CPU0:NC57C3-Vega-II5-53#
</code>
</pre>
</div>

Similarly you can use different ports in different speeds and breakout options. The table described above gives the combination details.

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

Currently we don't support the NC57-MPA-2D4H-S in the NC55-MOD-A line cards (base or scale).
