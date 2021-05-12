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

In the front, we have 100G ports (QDD cages, designed for high power optics) on the left part and 400G (QDD too) on the right.

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



----

<div class="highlighter-rouge">
<pre class="highlight">
<code> "Pre-existing [hw-module fib ipv4 scale internet-optimized] config has "  
 "been found. This feature isn't supported anymore and therefore ignored. "  
 "Please delete this config. "  </code>
</pre>
</div>

![host-optimized-default.png]({{site.baseurl}}/images/host-optimized-default.png){: .align-center}

