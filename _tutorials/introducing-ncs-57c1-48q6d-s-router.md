---
published: true
date: '2022-05-09 10:37 +0530'
title: Introducing NCS-57C1-48Q6D-S Router
author: Tejas Lad
excerpt: Introduction to NCS-57C1
tags:
  - iosxr
  - cisco
  - NCS5700
  - NCS5500
position: top
---
{% include toc icon="table" title="Introducing NCS-57C1-48Q6D-S Router" %}
## Introduction

This article has been written and reviewed by:

MIG PM/TME team

With IOS XR 7.5.2, we introduce one rack unit (1RU) fixed port routers in the Cisco NCS 5700 series. It is high-capacity, low power consuming router providing the following support and capabilities:
 - Up to 4T total port bandwidth (oversubscribed)
 - 4T forwarding capacity.
 - Total of 54 ports - 4 ports of 400G QSFP-DD, 2 ports of 4x100G QSFP-DD, 16 ports of 50G SFP+ (also support traffic speed of 10G, 25G, and 1G), 32 ports of 25G SFP+ (also support traffic speed of 10G and 1G)
 - Support for SFP, SFP+, SFP28, and QSFP28 optics
 - Synchronous Ethernet (SyncE) and PTP
 - Power supply redundancy (AC/DC)
 - MACSEC


## Naming Logic

![Screenshot 2022-05-09 at 3.56.50 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 3.56.50 PM.png)

NCS-57C1-48Q6D-S is the first product based on Qumran-2C chipset. The PIDs of the model will vary depending on the licencing model used. 

| PID               | Licensing Model                  |
|-------------------|----------------------------------|
| NCS-57C1-48Q6-SYS | Flexible Consumption Model (FCM) |
| NCS-57C1-48Q6D-S  | Perpetual/BAE                    |

Note: The show platform output will display the FCM Model

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ios#<mark>show platform</mark> 
Node              Type                     State                    Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        <mark>NCS-57C1-48Q6-SYS(Active)</mark> IOS XR RUN               NSHUT
0/FT0             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
0/FT1             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
0/FT2             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
0/FT3             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
0/FT4             FAN-1RU-PI-V2            OPERATIONAL              NSHUT
RP/0/RP0/CPU0:ios#
</code>
</pre>
</div>

## Future ready use cases

NCS-57C1-48Q6-SYS/NCS-57C1-48Q6D-S can be positioned in a very large variety of roles in the network due to its flexibility, compact form factor, combination of low and high rate ports and high-level forwarding capacity.

![Screenshot 2022-05-09 at 3.53.29 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 3.53.29 PM.png)

We will discuss the use cases in details in the below section.

## CCO documentation 

  - [Datasheet](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5700-series/ncs-57C1-fixed-chassis-ds.html). You can find the dimensions, power usage, PIDs
ports, supported standards in the same.
  - [Fixed systems White Paper](https://www.cisco.com/c/dam/en/us/products/collateral/routers/network-convergence-system-5500-series/ncs5500-fixed-platform-architecture-white-paper.pdf)
  - [Hardware Installation Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/hardware-install/b-ncs5700-hardware-installation-guide-fixed-port/m-ncs-5700-router-overview.html)

## Software

The product is launched with IOS-XR 7.5.2 and it will run XR7 and not XR 64bit. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ios#<mark>show version</mark> 
Cisco IOS XR Software, <mark>Version 7.5.2.12I LNT</mark>
Copyright (c) 2013-2021 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Mon Dec 20 20:45:07 UTC 2021
 Build Host   : iox-lnx-012
 Workspace    : /auto/iox-lnx-012-san2/prod/7.5.2.12I.DT_IMAGE/ncs5700/ws
 Version      : 7.5.2.12I
 Label        : 7.5.2.12I

cisco NCS5700 (D-1633N @ 2.50GHz)
cisco NCS-57C1-48Q6-SYS (D-1633N @ 2.50GHz) processor with 16GB of memory
ios uptime is 1 day, 23 hours, 12 minutes
NCS 57C1 Base Chassis, Flexible Consumption Need Smart Lic
</code>
</pre>
</div>

The system is based on a single Q2C, the operating system will be activated by default (no config required) in “Native mode”. From a security / integrity perspective, the NCS57C1 implements the latest secureboot features with TAM (Trust Anchor module) FPGA.










