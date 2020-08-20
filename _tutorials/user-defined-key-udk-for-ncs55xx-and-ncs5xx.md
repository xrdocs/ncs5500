---
published: true
date: '2020-08-20 23:04 +0530'
title: User Defined Key - UDK for NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document aims to deep dive into the concept of User Defined Keys on Cisco
  NCS55xx and NCS5xx
tags:
  - NCS5500
  - NCS500
  - ACL
  - UDK
  - User Defined Key
  - NCS55xx
position: hidden
---
{% include toc %}

## Introduction

In the previous technotes, ([ACL Packet Length Match](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/ "ACL Packet Length Match"),[ACL Fragment Match](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/ "ACL Fragment Match")) we have used the term User Defined Key - UDK many times and also saw it was compulsory to configure it for certain match criteria. In this technote, we will deep dive into the UDK concept and explore in details regarding the feature support.

## User-Defined Key - UDK

![Screenshot 2020-08-13 at 3.27.07 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-13 at 3.27.07 PM.png)
(Reference: NCS5500 deepdive)

As we already know, the NCS55xx and NCS5xx use either internal or external TCAM to perform the lookup and take defined action on each packet. Multiple features share the same TCAM resource in the hardware. Hence it needs to be utilized properly or else we are at a risk of running out of TCAM space. As default key definitions does not have enough space to include all qualifier/action fields, User-Defined Key (UDK) is needed. The space (key width) available for these key definitions is also constrained. A key definition specifies which qualifier and action fields are available to the ACL feature when performing the lookup. Not all available qualifier and action fields can be included in each key definition. ([Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html#id_100163 "Reference"))

The key definitions depend on the following attributes of the access-list:

| Attributes              | Details                 |
|-------------------------|-------------------------|
| Direction of attachment | Ingress or Egress       |
| Protocol type           | IPv4/IPv6/L2            |
| Compression level       | Uncompressed/Compressed |


## Advantages of using UDK

  - To include qualifier fields which are not included in the default TCAM key
  - To change the ACL mode from shared to unique to support a greater number of unique ACLs, unique counters, etc.
  - To reduce the size of the TCAM key (number of banks consumed)
  - For further information, please [refer](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html#id_100163 "refer")


## UDK Feature Support

  - A UDK can be defined globally or line card specific. The line card specific configuration will take  precedence over global configuration.
  - Only traditional or uncompressed ACL is supported.
  - Hybrid or Scaled ACL is not supported along with UDK.
  - A UDK definition will override the default key definition.
  - Only IPv4 and IPv6 keys in ingress direction are currently supported.
  - The IPv4 UDK supports a TCAM key size of 160 bits and 320 bits
  - The IPv6 UDK supports the size of 320 bits.
  - If the key defintion goes beyond the supported TCAM size, it will reject the ACL configuration. 


## When to use UDK ? 


Below table shows the frequently used qualifiers for IPv4 and IPv6. If the default TCAM key is set as Enabled, then the Qualifier field is enabled by default. 