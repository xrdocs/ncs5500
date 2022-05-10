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
position: hidden
---
{% include toc icon="table" title="Introducing NCS-57C1-48Q6D-S Router" %}
## Authors

This article has been written and reviewed by MIG PM/TME team

## Introduction

With IOS XR 7.5.2, we introduce one rack unit (1RU) fixed port routers in the Cisco NCS 5700 series. It is a high-capacity, low power consuming router providing the following support and capabilities:
 - Up to 4T total port bandwidth (oversubscribed)
 - 2.4T forwarding capacity.
 - Total of 54 ports - 4 ports of 400G QSFP-DD, 2 ports of 4x100G QSFP-DD, 16 ports of 50G SFP+ (also support traffic speed of 10G, 25G, and 1G), 32 ports of 25G SFP+ (also support traffic speed of 10G and 1G)
 - Support for SFP, SFP+, SFP28, and QSFP28 optics
 - Synchronous Ethernet (SyncE) and PTP
 - Power supply redundancy (AC/DC)
 - MACSEC


## Naming Logic

![Screenshot 2022-05-09 at 3.56.50 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 3.56.50 PM.png)

NCS-57C1-48Q6D-S is the first product based on Qumran-2C chipset. The PIDs will vary depending on the licencing model used. 

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

## Hardware Details

![Screenshot 2022-05-09 at 5.22.59 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 5.22.59 PM.png)

NCS57C1 is available in 1 RU Form Factor. It is 19’’ rack mountable and fits in standard 600 mm depth rack. It has 54 ports in the front panel. Out of which we have 4 400G multirate ports. 2 4x100G multirate ports. 16 50GE multirate ports and 32 25G multirate ports. It has an integrated RSP, timing and synchronization and LC complex in one box. It has one console and management port in the front panel along with all the data ports. All the timing ports are in the rear of the system. It has one USB 2.0 host port access, 1 RJ45 TOD port, 2 SMB connectors 1 1PPS and 1 10MHZ clock and one port for the GPS receiver. It has 2 Power supply slots. They are field replaceable with both AC and DC options. It operates in 1+1 active-active redundant mode. Each power supply has a built-in fan to cool the power supply. It supports front to back airflow. It has 5 Fan Units for cooling and proper airflow for the system with N+1 redundancy.

## Port Details

Let us see in details each block of ports:

### Ports 0-5

The platform supports 400G ZR/ZRP Optics supported only on top row i.e., ports 0,2,4. Ports 0,2,4,5 will support 400G Grey, 4x100G, 2x100G mode. Ports 1 and 3 will support only 4x100G and 2x100G Breakout. Ports 0-5 will support all the breakout options. Ports 0-5 will also support native 100G and 40G.

![Screenshot 2022-05-09 at 4.42.37 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 4.42.37 PM.png)

### Ports 6-21

Ports 6-21 will natively support 50G/25G/10/1G. No breakout is supported on these ports. No support for auto-negotiation as well.

![Screenshot 2022-05-09 at 4.44.20 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 4.44.20 PM.png)

### Ports 22-53

Ports 22-53 will natively support 25G/10/1G. No breakout is supported on these ports.No 1G auto-negotiation supported at FCS.

![Screenshot 2022-05-09 at 4.45.57 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 4.45.57 PM.png)


### Available Ports Summary

|                      | Native | Breakout | Max Ports | Comments                                               |
|----------------------|--------|----------|-----------|--------------------------------------------------------|
| 400G ZR/ZRP (native) | 3      | -        | 3         | Ports 0/2/4                                            |
| 400G Grey            | 6      | -        | 6         | Port 0-5 4x400G + 2(4x100G)                            |
| 100G                 | 6      | 24       | 24        | Ports 0-5 Native Support Ports 0-5 (4x100G) BO Support |
| 50G                  | 16     | -        | 16        | Ports 6-21                                             |
| 40G                  | 6      | -        | 6         | Ports 0-5                                              |
| 25G                  | 48     | 24       | 72        | Ports 6-53 Native Support Ports 0-5 (4x25G) BO Support |
| 10G                  | 48     | 24       | 72        | Ports 6-53 Native Support Ports 0-5 (4x10G) BO Support |
| 1G                   | 48     | -        | 48        | Ports 6-53                                             |

## Block Diagram

![Screenshot 2022-05-09 at 5.26.58 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 5.26.58 PM.png)

The entire port configurations is supported with a single Q2C ASIC. 400G ports i.e ports 0/2 and is connected to the ASIC via MACSEC PHY . Another 400G port is directly connected to the ASIC. We have another intermediate PHY which is connected in gearbox mode to two QSFPDD ports i.e ports 1 and 3 which will support 4x100G mode in each port. 16x50G ports are connected from optics to intermediate PHY to the ASIC. For these intermediate PHY will be configured in gearbox mode. These ports can also support 25G and 10G. In this mode, intermediate PHY will be configured in retimer mode.32x25G ports are connected from optics to PHY to ASIC. ASIC will be configured in 1x25G mode for each port and PHY will be configured in retimer mode. These ports can also support 10G and 1G. Control path is connected to the CPU. Timing chips are available to support SYNCE and PTP for class C performance.


## Port Assigment to ASIC core

![Screenshot 2022-05-09 at 7.55.57 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 7.55.57 PM.png)

The port mapping is pretty simple for this platform. As we have single NPU and single core all the interfaces will be mapped to a single core on the NPU. The important thing to highlight from the above output is the default speed of the interfaces when the platform boots up. As you can see ports 0/2/4 and 5 comes up as 400G whereas ports 1 and 3 comes up as 100G. So we need CLI command to change the speed of these ports. Ports 6 to 21 will come up as 50 gig and the rest 32 ports will come up as 25G when platform the first boots up. In these ports we do not need any CLI command to change the speed. It will automatically detect the optics and accordingly bring up the port.

![Screenshot 2022-05-09 at 7.57.13 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 7.57.13 PM.png)

## MDB profile 

NCS57C1 will also support the MDB profiles. At FCS we will support profile L3MAX and in upcoming releases we will support L2MAX along with L3MAX. The platform will not support the scale MDB profiles as we do not have eTCAM in this platform. Below is the MDB profile output on the platform

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:ios#show controllers fia diagshell 0 "mdb info" location 0/RP0/CPU0
Node ID: 0/RP0/CPU0

R/S/I: 0/0/0 
=============================
| MDB Profile               |
|   <mark>MDB profile: l3max      |</mark>
|   MDB profile KAPS cfg: 2 |
=============================
</code>
</pre>
</div>

## MACSEC 

![Screenshot 2022-05-09 at 8.03.11 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 8.03.11 PM.png)

As mentioned earlier ports 0/2/4 will be supporting MACSEC through PHY. Below table summarises different modes and combinations which can be used for MACSEC support. 

| Port Speed/Breakout Mode | MACSEC Support |
|--------------------------|----------------|
| Native 400G              | Yes            |
| Native 100G              | Yes            |
| Native 40G               | Yes            |
| 400G/4x100G              | Yes            |
| 400G/2x100G              | Yes            |
| 100G/4x25G               | Yes            |
| 40G/4x10G                | Yes            |

Note: Ports 1,3,5 and 6-53 does not support MACSEC.
{: .notice--info}

## TIMING

![Screenshot 2022-05-09 at 9.52.24 PM.png]({{site.baseurl}}/images/Screenshot 2022-05-09 at 9.52.24 PM.png)

- From timing perspective, the platform is capable of supporting class C timing
- SyncE and PTP is also supported on the breakout options.
- 1G mode support Class A. 
- AT FCS we will be supporting all the PTP profiles with IPv4. 
- With IPv6 we will support in future releases. 
- BITS is not supported at FCS.

## Power Consumption

Below is the power consumption of the router

|                      | NCS-57C1-48Q6D-S                                   |
|----------------------|----------------------------------------------------|
| Power without Optics | Typical:340W (25C) Max: 488W (40C)                 |
| Power with Optics    | Typical:550W (25C) Max:690W (40C)                  |
| Comments             | 48x SFP-25G-SR, 3x QDD-400G-ZRP-S, 3x QDD-400G-DR4-S |


## References

For other recently launched products and understanding their architecture please check the below:

- [Introducing the NCS57C3 Routers](https://xrdocs.io/ncs5500/tutorials/introducing-ncs57c3-mod/)
- [Introducing the NCS57B1 Routers](https://xrdocs.io/ncs5500/tutorials/introducing-ncs57b1-routers/)
