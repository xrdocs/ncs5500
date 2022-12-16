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

According to [Cisco's Annual Internet Report](https://www.cisco.com/c/en/us/solutions/collateral/executive-perspectives/annual-internet-report/white-paper-c11-741490.html), the number of devices connected to IP networks will be more than three times the global population by 2023. There will be 3.6 networked devices per capita by 2023, up from 2.4 networked devices per capita in 2018. There will be 29.3 billion networked devices by 2023, up from 18.4 billion in 2018. According to Gartner, by 2025 more than 75% of the computing data will need to be analyzed, processed, and stored closer to the end users. New paradigms such as 5G introduction, rapid growth in video traffic, and proliferation of IoT and cloud services model require unprecedented flexibility, elasticity and scale from the network. Increasing bandwidth demands and decreasing ARPU is putting pressure to reduce network cost. At the same time, services need to be deployed faster and more cost-effectively to stay competitive. The metro network becomes the key to this transformation where we see services getting terminated and end users looking for better experiences. In this article, we will discuss latest trends in the metro architecture and how communication service providers (CSPs) can design their network for optimized value proposition.  

![Screenshot 2022-12-13 at 1.40.09 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 1.40.09 PM.png)

## Metro Architecture Shift

![Screenshot 2022-12-13 at 1.52.45 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 1.52.45 PM.png)

We are seeing a [major architecture shift](https://blogs.cisco.com/sp/catching-the-next-wave-of-innovation-in-routing-and-automation) in the way the new metro fabric is evolving. Applications (8K Video, Gaming, AR/VR) are becoming more dynamic requiring differentiated user experiences and optimizing distributed compute and decomposed network resources in places where power and space are constrained. These trends have driven the evolution of our product portfolio with enhanced software capabilities and compact form-factors. As we look into the future, the trends will require even more accelerated innovation.

## Need for architecture transistion

![Screenshot 2022-12-13 at 5.23.19 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 5.23.19 PM.png)

In the past CSPs used to build siloed networks for their mobile, enterprise and residential networks. But this creates challenges for them to maintain their networks efficiently. They need to move to a single transport network which allows them to move to faster service delivery. The [Converged Transport architecture](https://www.cisco.com/c/en/us/solutions/collateral/service-provider/converged-sdn-transport/at-a-glance-c45-744218.html) uses advanced features and technology to help CSPs design and migrate to a network prepared to scale to meet the stringent bandwidth and performance demands of their customers. At the fundamental level, it drives simplification into the CSP’s network and operations.

Implementing the Converged Transport architecture will allow CSPs to realize the following business benefits:

- Reduced operational complexity for network management
- Increased revenue with a service-focused, service-centric network
- Improved time to market for new value-added services
- Optimized utilization of fiber capacity
- Decreased operational and capital expenditures associated with the network


## Evolved Metro Use cases

Below are some of the use cases which we see will drive the evolution of the metro architecture as well as the products.

![Screenshot 2022-12-13 at 5.36.45 PM.png]({{site.baseurl}}/images/Screenshot 2022-12-13 at 5.36.45 PM.png)

## NCS5500/5700 Portfolio Evolution for supporting the converged metro

The Cisco [NCS 5500/5700](https://www.cisco.com/c/en/us/products/routers/network-convergence-system-5500-series/index.html) Series is an industry-leading portfolio developed to handle massive traffic growth. It is designed for operational simplicity and for efficiently meeting the scaling needs of large enterprises, web, utilities and service providers. It comprises a range of products in fixed (1RU,2RU,3RU) and Modular (4,8,16 slots) form factors.

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


## NCS5500/NCS5700 Capabilities to design the next generation Metro

Powered by Cisco IOS-XR, NCS5500/5700 has all the capabilities to design the next-gen metro. They provide cost structures and power efficiency in a fixed systems as well as flexibility of a modular system in terms of redundancy options, upgradability, higher radix, and interface mixing. Below are the top capabilities we think are needed to build a successful network.

| Metro Network Capabilities              | Support                                                                                                                                                                                                                |
|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| High Speed Transport                    | Supports [400G ZR/ZRP](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/73x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-73x/m-zr-zrp.html) <<br>Capability of dense 100G/400G in small 1RU form factors <br>Ability to support low speeds from 1G/10G/25G/40G along with 100G/200G/400G                                           |
| Packet and Optical Convergence          | Unleash transformative operational efficiency with [Cisco Routed optical Network](https://www.cisco.com/c/en/us/solutions/service-provider/routed-optical-networking/index.html) -aka RON <br>Achieve Network efficiency and Service Profitability <br>Lower TCO with Simple Architecture                           |
| Network Slicing using Segment Routing   | Full support for [Segment Routing/SRv6](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/segment-routing/76x/b-segment-routing-cg-ncs5500-76x.html) Features <br>Capability to impose 26 usids in a single pass <br>Support for Flex Algo/SRTE/Ti-LFA<br>MicroLoop Avoidance and SR-PM plus many more features           |
| Service deployment using EVPN Overlay   | Support for next-gen [EVPN Features](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/vpn/76x/b-l2vpn-cg-ncs5500-76x/configure-evpn-irb.html) <br>EVPN-VPWS,EVPN-ELAN, EVPN Etree <br>EVPN IRB Anycast Gateway <br>EVPN Single home and Multihome <br>EVPN Single Active, Active-Active, Port-Active Capabilities |
| Network Security with Trustworth Platforms                        | Built-in [hardware security](https://www.cisco.com/c/en/us/about/trust-center/technology-built-in-security.html) along with software security in IOS-XR <br>Secure-boot with Cisco TAM chip<br>DDoS protection<br>Malware protection<br>Anti-Theft protection<br>Secure app-hosting environment          |
| IP-SLA                                  | <br>Network performance monitoring with rich set of [IP-SLA features](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/system-monitoring/76x/b-system-monitoring-cg-ncs5500-76x/m-ipsla-twamp-ncs5500.html) <br>TWAMP/TWAMP-Lite<br>                                                                                                                       |
| Ability to terminate non-Ethernet ports | Transparently transfer ODU/SONET/SDH/Eth/FC packets over PSN using [PLE](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-5500-series/network-con-5500-series-ds.pdf)                                                                                                                                                 |
| Flexible Resource Carving               | Support for different [MDB profiles](https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/) to cater multiple use cases                                                                                                                                                         |
| Quality of Service                      | Rich ingress and egress QoS features <br>Support for Policing/Shaping/WFQ/WRED etc <br>Enhance QoS scale with [Egress Traffic Manager](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/qos/74x/b-qos-cg-ncs5500-74x/m-configure-egress-traffic-management.html#Cisco_Concept.dita_e0ac2096-7453-4a18-a2a7-3a9c00cc46d6) implementation                                                            
| MACsec | Layer 2 security with full [MACsec](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/76x/b-system-security-cg-ncs5500-76x/implementing-macsec-encryption.html) support |
| Timing                                  | <br>Class C timing support for low latency applications                                                                                                                                                                |
| Telemetry                               | Support for Model Driven as well as AI driven Telemetry                                                                                                                                                                |
| Automation                              | Flexible Automation with [Cisco Crossworks Network Automation](https://www.cisco.com/c/en/us/products/cloud-systems-management/crosswork-network-automation/index.html)                                                                                                                                                                              |
| Programmability      | Drive Network Automation with Using [Programmable models](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/programmability/76x/b-programmability-cg-ncs5500-76x.html) <br> Support for Yang Models <br> Support for netconf and grpc protocols <br> Automation scripts |
| Sustainability                          | Reduce environmental impacts with systems delivering lower power consumption and by integrating automation tools to reduce manual and onsite operation <br>[Cisco Sustainability Report](https://www.cisco.com/c/dam/m/en_us/about/csr/esg-hub/_pdf/purpose-report-2021.pdf)                            |


## Conclusion

Going forward the challenge for CSPs will not just be to deliver services, but also to deliver experience. With metro network as the center of gravity along edge/cloud convergence combined with the broadband, it is crucial for the CSPs to seriously invest in upgrading their network. CSPs need to think not only about delivering the experience but also about ways to monetize it. Cisco is committed to be a trusted partner for the same and is equipped with the capabilities to help CSPs build a simple, scalable, programmable and sustainable network.

## References

- [NCS5500/5700 Resources](https://xrdocs.io/ncs5500/)
- [Metro Fabric High Level Design](https://xrdocs.io/design/blogs/2018-10-01-metro-fabric-hld/)
- [Implementation Guide](https://xrdocs.io/design/blogs/latest-converged-sdn-transport-ig)

