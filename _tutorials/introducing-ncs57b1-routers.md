---
published: true
date: '2021-05-11 22:04 +0200'
title: Introducing NCS 57B1 Routers
author: Nicolas Fevrier
excerpt: Introduction of the new NCS 57B1 Routeurs
position: hidden
---
{% include toc icon="table" title="NCS 57B1 Routers" %} 

## Intro

With IOS XR 7.3.1, we introduced multiple software features ([https://xrdocs.io/ncs5500/tutorials/iosxr-731-innovations/](https://xrdocs.io/ncs5500/tutorials/iosxr-731-innovations/)) but new hardware. Among them, the NCS 57B1 series.

These two new routers are the NCS57B1-6D24 and NCS57B1-5DSE, both  1RU systems with a mix of 100G and 400G ports and powered by a Broadcom Jericho2 NPU.  
They can be used in multiple places in the network, like aggregation (100G to 400G), 5G (class-C capable), internet peering, core, ...  
If it can offer 10G breakout through QSA and break-out cables, this router is optimized for 100G and 400G interfaces.

## Videos

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/MyqmIlozL8M" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

And since you didn't ask for it, the unboxing :))

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/HRKhKuMAy-g" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

## Naming logic

To understand where the product names NCS57B1-6D24 and -5DSE come from:

![naming.png]({{site.baseurl}}/images/naming.png){: .align-center}


## Product description

You can find the datasheet on Cisco's website:  
[https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-744698.html](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/datasheet-c78-744698.html)  
You can find a lot of details on the part numbers, licenses, standards compliance, and tons of details on the products.

We differentiate:  
- the "base" system (Jericho 2 without eTCAM) offering 24x 100G and 6x 400G ports: **NCS57B1-6D24**

![6D24 plongeant.png]({{site.baseurl}}/images/6D24 plongeant.png){: .align-center}

- the "scale" system (Jericho 2 with OP2 eTCAM) offering 24x 100G and 5x 400G ports: **NCS57B1-5DSE**

![5DSE plongeant-good.jpg]({{site.baseurl}}/images/5DSE plongeant-good.jpg){: .align-center}

The difference between the two versions is simply the presence of an eTCAM and for the system equipped with this additional resource, we have one port QSFP-DD 400G left.  
The connection to the eTCAM is made through fabric ports for the routes and classifiers, but we are using "NIF" ports for the statistics, hence the impact on the 400G port density.

### Hardware

![dimensions.png]({{site.baseurl}}/images/dimensions.png){: .align-center}

From the back, you find:  
- two power supply modules (AC or DC) 2kW offering 1+1 redundancy (PSU2KW-ACPI or PSU2KW-DCPI). Note we don't support the mix of AC and DC, and the cooling is only front to back today.
- 6 fan trays (NC57-B1-FAN1-FW) offering 5+1 redundancy.

![]({{site.baseurl}}/images/6D24%20back%20plat.jpg){: .align-center}

In the front, we have 100G ports (QDD cages, designed for high power optics) on the left part and 400G (QDD too) on the right. These cages will permit the use of 100G ZR optics in the upper row (the exact amount of optics is still under validation and will be updated later) and 400G ZR/ZR+ in all the ports on the right.

![100G-ports.png]({{site.baseurl}}/images/100G-ports.png){: .align-center}

![400G-ports.png]({{site.baseurl}}/images/400G-ports.png){: .align-center}

Finally, on the right end, we have the timing ports and management ports.  
- RJ45 for TOD
- SMB connectors (1PPS and 10Mhz clock)
- One antenna port for GPS

![timing-mgt-new.png]({{site.baseurl}}/images/timing-mgt-new.png){: .align-center}

The NCS 57B1 routers are Class-C timing capable, on all ports.

### NPU

These routers are "system on a chip" or SoC. All ports are connected to a single chipset (via reverse gear boxes, we will describe it further, later with the block diagram). That makes the design very simple and efficient in term of latency, performance and power consumption.  
At the heart, we have the Broadcom Jericho 2 NPU. It offers 4.8 Tbps and 2BPPS of forwarding capacity.  
Like it's predecessors using in NCS 5500 platforms, it's a VOQ-only forwarding architecture with ingress hybrid packet buffering. Hybrid here means the packets can be stored in a 32MB on-chip memory or in an 8GB external HBM in case of queue congestion.

![J2-2.png]({{site.baseurl}}/images/J2-2.png){: .align-center}

The NPU uses 50Gbps SERDES to connect to the Reverse Gear Boxes

### Block Diagrams

NCS57B1-6D24 and NCS57B1-5DSE:

![diagram-6D24.png]({{site.baseurl}}/images/diagram-6D24.png){: .align-center}

![diagram-5DSE.png]({{site.baseurl}}/images/diagram-5DSE.png){: .align-center}

From an architecture point of view, very few differences between the two platforms. Some interfaces links being used to connect to the statistic part of the OP2 eTCAM, hence the different 400G port density.  
Jericho 2 ASIC is connected to PHY chipsets operated in reverse gear boxes to connect to the 100G interfaces or switching mode to connect to the 400G interfaces.  

## Software

The NCS571B are both introduced with IOS XR 7.3.1, that's the minimum release to support them.  
Since it's a standalone platform, it operates by default in "Native Mode", no configuration is required to enable that mode.  

Note: this system is using the newest version of IOS XR named XR7. XR7 is the latest evolution of the operating system and from feature and scale is similar to XR64bit used in other NCS5500 product. The main noticeable difference will be in the upgrade process based on Golden ISO images and the "install replace" approach instead of the "install add / install activate" methodology.  
Take a look at the setup guide for mode details.

## Ports configuration

It's possible to use the router in multiple roles, like aggregation of 100G and 400G ports for example.  
It offers 24 ports 100G and 5 or 6 ports 400G. So it's possible to imagine cases with 2.4Tbps to the clients in 100G and 2.4Tbps to the core in 400G. But nothing prevents from breaking some or all of the 400G ports in 4x100G, to build a 32x 100G + 4x 400G systems. It's very flexible.

Keep in mind some basic concept here: you can not break a 400G interfaces and expect to connect it to existing 100G SR4, LR4 or CWDM4. The 100G port facing the breakout cable must be "1-lambda" optics like DR/FR.

![break-out-rules.png]({{site.baseurl}}/images/break-out-rules.png){: .align-center}



