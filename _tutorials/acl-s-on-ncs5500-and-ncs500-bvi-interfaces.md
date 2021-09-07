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

![Screenshot 2021-09-07 at 3.52.22 PM.png]({{site.baseurl}}/images/Screenshot 2021-09-07 at 3.52.22 PM.png)

Before we move on the ACL feature, let us do a quick recap of the BVI interface and understand its use cases. The [**BVI**](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/interfaces/71x/configuration/guide/b-interfaces-hardware-component-cg-ncs5500-71x/m-adhoc-fretta-irb-unicast.html) is a virtual interface within the router that acts like a normal routed interface. BVI provides link between the bridging and the routing domains on the router. The BVI does not support bridging itself, but acts as a gateway for the corresponding bridge-domain to a routed interface within the router.
It is associated to a bridge group using the routed interface bvi command. 

Bridge-Domain: It is a layer 2 broadcast domain
{: .notice--info}

### Main Use cases

  - Interconnect bridged and routed networks
  - Preserve network addresses
  - Bridge local traffic for efficient network performance

## Configuring ACL and attaching to BVI interface

Configuration and attachment of an ACL over BVI interface is similar to regular ACLs and attachment to a physical interface. Let us check out the same.

### Ingress V4 ACL

### Ingress V6 ACL

### Egress V4 ACL

### Egress V6 ACL






  
  

