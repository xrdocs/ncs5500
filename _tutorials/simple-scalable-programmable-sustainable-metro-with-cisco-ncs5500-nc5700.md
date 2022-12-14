---
published: true
date: '2022-12-13 10:49 -0800'
title: 'Simple, Scalable, Programmable & Sustainable Metro with Cisco NCS5500/NC5700'
author: Tejas Lad
excerpt: This document covers the latest trends in metro network architecture
tags:
  - iosxr
  - cisco
  - NCS5500
  - NCS5700
  - Metro
  - Metro Fabric
position: hidden
---
{% include toc icon="table" title="Simple,Scalable, Programmable & Sustainable Metro with Cisco NCS5500/5700" %}

## Introduction

As per [Cisco's Annual Internet Report](https://www.cisco.com/c/en/us/solutions/collateral/executive-perspectives/annual-internet-report/white-paper-c11-741490.html), the number of devices connected to IP networks will be more than three times the global population by 2023. There will be 3.6 networked devices per capita by 2023, up from 2.4 networked devices per capita in 2018. There will be 29.3 billion networked devices by 2023, up from 18.4 billion in 2018. As per Gartner, by 2025 more than 75% of the computing data needs to be analyzed, processed, and stored near to the end users. New paradigms such as 5G introduction, video traffic continuous growth, IoT proliferation and cloud services model require unprecedented flexibility, elasticity and scale from the network. Increasing bandwidth demands and decreasing ARPU put pressure on reducing network cost. At the same time, services need to be deployed faster and more cost effectively to stay competitive. The metro network becomes the key to this transformation where we see services getting terminated and end users are looking for better experiences. In this article, we will discuss latest trends in the metro architecture and how to communication service providers (CSPs) can design their network for optimised value proposition.  

![Screenshot 2022-12-13 at 1.40.09 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 1.40.09 PM.png)

## Metro Architecture Trends

![Screenshot 2022-12-13 at 1.52.45 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 1.52.45 PM.png)

We are seeing a [major architecture shift](https://blogs.cisco.com/sp/catching-the-next-wave-of-innovation-in-routing-and-automation) in the way the new metro fabric is evolving. Applications (8K Video, Gaming, AR/VR) are becoming more dynamic requiring differentiated user experiences and optimizing distributed compute and decomposed network resources in places where power and space are constrained. Below key trends has driven the evolution of our routing portfolio with software capabilities and compact form-factors. As we look to the future, the trends will require even more accelerated innovation.

## Need for architecture transistions

![Screenshot 2022-12-13 at 5.23.19 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 5.23.19 PM.png)

In past CSPs used to build siloed networks for their mobile, enterprise and residential networks. But this is not helping the CSPs to maintain their networks efficiently. They need to move to a single transport network which allows them to move to faster service delivery. The C[onverged Transport architecture](https://www.cisco.com/c/en/us/solutions/collateral/service-provider/converged-sdn-transport/at-a-glance-c45-744218.html) uses advanced features and technology to help CSPs design and migrate to a network prepared to scale to meet the stringent bandwidth and performance demands of their customers. At the fundamental level, the Converged SDN Transport architecture drives simplification into the CSP’s network and operations.

Implementing the Converged SDN Transport architecture will allow CSPs to realize the following business benefits:

- Reduced operational complexity for network management
- Increased revenue with a service-focused, service-centric network
- Improved time to market for new value-added services
- Optimized utilization of fiber capacity
- Decreased operational and capital expenditures associated with the network


## Evolved Metro Use cases

Below are some of the use cases which we see will drive the evolution of the metro architecture as well as the products.

![Screenshot 2022-12-13 at 5.36.45 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 5.36.45 PM.png)

## NCS5500/5700 Portfolio Evolution for supporting the converged metro

The Cisco [NCS 5500/5700](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html) Series is an industry-leading portfolio developed to handle massive traffic growth. It is designed for operational simplicity and to efficiently meet the scaling needs of large enterprises, web, Utilities and service providers. It has a range of products from fixed platforms (1RU,2RU,3RU) to Modular Chassis (4,8,16 slots).

![Screenshot 2022-12-13 at 6.02.18 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 6.02.18 PM.png)


### Portfolio details

#### Fixed

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
| NCS-57D2-18DD-S     | 2           | 66 Ports: 2x 400G QSFP-DD  +<br> <br>                 16x 400GQSFP-DD / 64x 100G QSFP-DD                                                                                                        |

#### Modular Line Cards

| Product ID        | Description                                                                                               |
|-------------------|-----------------------------------------------------------------------------------------------------------|
| NC55-32T16Q4H-A   | 32x1/10GE SFP/SFP+ <br><br>16x10/25GE SFP+/SFP28 <br><br>4x40/100GE QSFP+/QSFP28 <br><br>(oversubscribed) |
| NC55-MOD-A(-SE)-S | Fixed:12x1/10G SFP/SFP+2x40G QSFP+ <br><br>2X MPA:2x400G or 12x50G                                        |
| NC55-36X100-A-SE  | 36x100/40G                                                                                                |
| NC57-24DD         | 24x400G QSFPDD                                                                                            |
| NC57-18DD-SE      | 18x400G QSFPDD or 30x200/100G QSFPDD                                                                      |
| NC57-36H6D-S      | 24x100G + 12 flex ports (6x400GE or 12x200GE/100GE)                                                       |
| NC57-36H-SE       | 36x100G QSFPDD                                                                                            |
| NC57-MOD-S        | Fixed:2x400G QSFPDD+ 8x10/25/50G SFP56 <br><br>2X MPA:2x400G or 12x50G                                    |






