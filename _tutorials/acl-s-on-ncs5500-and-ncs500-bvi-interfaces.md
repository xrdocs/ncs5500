---
published: true
date: '2021-09-07 12:37 +0530'
title: ACL's on NCS5500 and NCS500 BVI Interfaces
author: Tejas Lad
excerpt: This document will introduce the behaviour of ACL on BVI Interfaces
tags:
  - iosxr
  - NCS5500
  - NCS500
  - ACL
  - Access List
  - BVI
position: hidden
---
{% include toc icon="table" title="ACL's on NCS5500 and NCS500 BVI Interfaces" %} 

## Introduction

In the [previous article](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/), we introduced the ACL enhancements on NCS5500 based on J2 chipsets. In this article, we will introduce the ACL implementation on NCS5500 w.r.t to BVI interfaces. We will cover all the platforms based on J/J+ and J2. 

## Quick Recap: Bridged Virtual Interface - BVI 

![Screenshot 2021-09-09 at 12.07.56 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-09 at 12.07.56 PM.png)

Before we move to the ACL features, let us do a quick recap of the BVI interface and understand its use cases. The [**BVI**](https://www.cisco.com/c/en/us/support/docs/lan-switching/integrated-routing-bridging-irb/17054-741-10.html) is a virtual interface within the router that acts like a normal routed interface. BVI provides link between the bridging and the routing domains on the router. The BVI does not support bridging itself, but acts as a gateway for the corresponding bridge-domain to a routed interface within the router. Bridge-Domain is a layer 2 broadcast domain
It is associated to a bridge group using the routed interface bvi command. 


### Main Use cases

  - Interconnect bridged and routed networks
  - Preserve network addresses
  - Bridge local traffic for efficient network performance

BVI provides a much more flexible solution for bridging and routing
  
| Scenarios                                                    | Supported by BVI |
|--------------------------------------------------------------|------------------|
| Communication of multiple interfaces in same BD              | Yes              |
| Communication of multiple interfaces in different BD         | Yes              |
| Communication between Bridged interface and Routed interface | Yes              |
  
  

## Configuring ACL and attaching to BVI interface

Configuration and attachment of an ACL over BVI interface is similar to regular ACLs and attachment to a physical interface. Let us check out the same.

### Ingress V4 ACL

### Ingress V6 ACL

### Egress V4 ACL

### Egress V6 ACL


## References

- https://www.cisco.com/c/en/us/support/docs/lan-switching/integrated-routing-bridging-irb/17054-741-10.html
- https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/71x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-71x/m-adhoc-fretta-irb-unicast.html
