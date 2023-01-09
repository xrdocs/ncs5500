---
published: true
date: '2023-01-09 10:16 -0800'
title: Introducing NCS-57D2-18DD-SYS
author: Tejas Lad
excerpt: Introduction to NCS-57D2
tags:
  - iosxr
  - cisco
  - NCS5500
  - NCS5700
  - metro
  - NCS57-D2-18DD
position: hidden
---
{% include toc icon="table" title="Introducing NCS-57D2-18DD-SYS" %}

## Introduction

Latest trends in the [metro architecture](https://xrdocs.io/ncs5500/tutorials/simple-scalable-programmable-sustainable-metro-with-cisco-ncs5500-nc5700/) have driven the evolution of our [product portfolio](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html) with enhanced software capabilities and compact form-factors. The Cisco NCS 5500/5700 Series offers industry-leading 100 GbE and 400 GbE port density to handle massive traffic growth. It is designed for operational simplicity and to efficiently meet the scaling needs of large enterprises, web, and service providers. With [IOS-XR 7.8.1](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/78x/release/notes/b-release-notes-ncs5500-r781.html#Cisco_Concept.dita_f911b3f4-9c37-46b7-b0b4-c151641ff919), we have introduced the latest addition to our portfolio: [NCS57-D2-18DD-SYS](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs-57d2-18dd-fixed-chassis-ds.html). In this article, we will discuss the features and its use cases.

## NCS5500/5700 Fixed Portfolio

Before deep-diving into the NCS57-D2, let us have a quick refresher of the NCS5500/5700 fixed portfolio. 

| Product ID          | Form Factor | Description                                                                                                                                                                                     |
|---------------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| NCS-5501(SE)        | 1           | Base: 48x 1/10G SFP + 6x 40/100G QSFP <br><br>Scale : 40x 1/10G SFP + 4x 40/100G QSFP                                                                                                           |
| NCS-55A1-36H(-SE)-S | 1           | 36x 40/100G QSFP                                                                                                                                                                                |
| NCS-55A1-24H        | 1           | 24x 40/100G QSFP                                                                                                                                                                                |
| NCS-55A1-24Q6H-S    | 1           | 24x 1/10G SFP + 24x 1/10/25G SFP<br>+ 6x 40/100G QSFP                                                                                                                                           |
| NCS-55A1-48Q6H      | 1           | 48x 1/10/25G SFP + 6x 40/100G QSFP                                                                                                                                                              |
| NCS-55A2-MOD        | 2           | 24x 1/10G SFP + 16x 1/10/25G SFP<br>+ 2x 400G MPAs <br><br>MPA 4x 100G QSFP<br>MPA 2x 100/200G DWDM CFP2<br>MPA 2x 100G QSFP + 1x 100/200G DWDM CFP2<br>MPA 12x 1/10G SFP                       |
| NCS-57B1-6D24/5DSE  | 1           | Base: 24x 100G QSFPDD + 6x400G QSFPDD <br> <br>Scale:24x 100G QSFPDD + 5x400G QSFPDD                                                                                                            |
| NCS-57C1-48Q6D-S    | 1           | 32x SFP28 + 16x SFP56 +<br>2x QSFP-DD(4x100G) + 4x QSFP-DD (400G)                                                                                                                               |
| NCS-57C3-MOD(S)-SYS | 3           | 48x 1/10/25G + 8x or 4x port 40/100G + <br><br>MPAs:<br>2x400G or 4x 100G MPA<br>MPA 4x 100G QSFP<br>MPA 2x 100/200G DWDM CFP2<br>MPA 2x 100G QSFP + 1x 100/200G DWDM CFP2<br>MPA 12x 1/10G SFP |
| **NCS-57D2-18DD-S**     | 2           | 66 Ports: 2x 400G QSFP-DD  +<br> <br>                 16x 400GQSFP-DD / 64x 100G QSFP-DD                                                                                                        |

## NCS57D2 Product Specifications

**Front view**

![Screenshot 2023-01-09 at 11.02.11 AM.png]({{site.baseurl}}/images/Screenshot 2023-01-09 at 11.02.11 AM.png)

NCS57-D2 is a compact 2RU chassis purpose built for dense 400G aggregation. It offers 7.2 Terabits of 400GE/100GE optimized forwarding capacity, high power efficiency, QSFP-DD optics, deep packet buffering combined with next-generation security, automation, telemetry, segment routing, EVPN, and Equal-Cost Multipathing (ECMP) capabilities making it suitable for high capacity metro applications. The industry-leading Cisco IOS XR pushes the scalability of the hardware even further through the use of flexible [MDB profiles](https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/) (modular database) that allows ASIC and TCAM resources to be reconfigured as per the network use cases.

### Physical Interfaces

**Fixed ports**

- Integrated 40/100/400 Gigabit Ethernet support
- 18x 400G ports with QSFP-DD
- 66x 100G ports with QSFP-DD
- All ports support QSFP-28 and QSFP+
- Breakout options supported for 10G/25G/100G
- Support for ZR/ZR+ Coherent optics

**Other port**

![]({{site.baseurl}}/images/Screenshot%202023-01-09%20at%2011.18.26%20AM.png)

- RJ-45 ports for console and management
- 1 Pulse Per Second (PPS) in/out, 10 MHz in/out, Time of Day (ToD)
- Internal Global Navigation Satellite System (GNSS) with antenna port
- 1 USB port

### Physical Specifications

**Dimensions**

- Height: 2RU 3.45” in. (8.76 cm)
- Width: 17.60 in. (43.94 cm)
- Depth: 23.62 in. (59.99 cm)

**Weight**

- 53 lb (24 kg) including fans and PSUs

**Power**

-Typical: 500W without optics
-Power handling of each quad = 43W

Note: Quad is a group of 4 ports.
{: .notice--info}

## Naming Logic 

![]({{site.baseurl}}/images/Screenshot%202023-01-09%20at%2011.29.23%20AM.png)

NCS-57D2 is built with J2C+ chipset. The PIDs will vary depending on the licencing model used. 

| PID               | Licensing Model                  |
|-------------------|----------------------------------|
| NCS-57D2-18DD-SYS | Flexible Consumption Model (FCM) |
| NCS-57D2-18DD-S  | Perpetual/BAE                    |

Note: The show platform output will display the FCM Model

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:BGL-CB#show platform 
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        NCS-57D2-18DD-SYS(Active) IOS XR RUN               NSHUT
0/PM0             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/PM1             PSU2KW-ACPI              OPERATIONAL              NSHUT
0/FT0             NC57-D2-FAN-FW           OPERATIONAL              NSHUT
0/FT1             NC57-D2-FAN-FW           OPERATIONAL              NSHUT
0/FT2             NC57-D2-FAN-FW           OPERATIONAL              NSHUT
0/FT3             NC57-D2-FAN-FW           OPERATIONAL              NSHUT
0/FB0             NC57-D2-FAN-FW           OPERATIONAL              NSHUT
RP/0/RP0/CPU0:BGL-CB#
</code>
</pre>
</div>

## NCS-57D2 Use cases

### High End Aggregation


