---
published: true
date: '2023-06-23 13:28 -0700'
title: Filtering IPv6 Packets with Extension Headers on NCS5500/5700
author: Tejas Lad and Bala Murali Krishna Sanka
excerpt: >-
  This document will show the NCS5500/5700 capability to filter IPv6 packets
  with extension headers
tags:
  - iosxr
  - cisco
  - NCS5500
  - NCS5700
  - IPv6
  - ACL
  - IPv6 EH
position: hidden
---
{% include toc icon="table" title="Table of Contents" %}

|Tejas Lad, Technical Marketing Engineer|  
|Bala Murali Krishna Sanka, Technical Marketing Engineer|

## Introduction

In our previous [tech-note](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/), we saw how User Defined Fields - UDF can provide advanced filtering capabilities on NCS5500/5700. We saw examples of how different fields can be matched and used to filter traffic. In this tech-note, we will see how IPv6 packets with Extension Header can also be filtered using UDF. 

## What is IPv6 Extension Headers ?

Before filtering the packets with Extension Header (EH), let us try to understand what is extension header. Let us understand the implications of the presence of extension headers  the native IPv6 traffic forwarding performance of network devices. ([Reference](https://www.cisco.com/en/US/technologies/tk648/tk872/technologies_white_paper0900aecd8054d37d.html)). IPv6 has two distinct types of headers: **Main/Regular IPv6 Header** and **IPv6 Extension Headers**. The main IPv6 header is equivalent to the basic IPv4 shown below. An IPv6 basic header is 40 bytes long. Each IPv6 packet has basic header that provides basic packet forwarding information, which is needed for each devices in the forwarding path.

![Screenshot 2023-06-23 at 3.17.43 PM.png]({{site.baseurl}}/images/Screenshot 2023-06-23 at 3.17.43 PM.png)

### Comparison to options field in IPv4 header

The options field in the IPv4 header is very important which is used to convey additional information on the packet or on the way it should be processed. Unless instructed otherwise, network devices must process the options in the IPv4 header. The processing of most header options pushes the packet into the slow path leading to a forwarding performance hit. This capability has been preserved in IPv6 packet. During the development of IPv6, the impact of IPv4 Options on performance was taken into consideration. The functionality of options is removed from the main header and implemented through a set of additional headers called Extension Headers. The main header remains fixed in size (40 bytes) while customized EHs are added as needed. 

### Every IPv6 packet carry one extension header?

As per [RFC 2460](https://datatracker.ietf.org/doc/html/rfc2460) in IPv6,   optional internet-layer information is encoded in separate headers that may be placed between the IPv6 header and the upper-layer header in a packet.  There are a small number of such extension headers, each identified by a distinct Next Header value. An IPv6 packet may carry zero, one, or more extension headers, each identified by the Next Header field of the preceding header. 

Note: RFC 2460 is now obselete and it is taken over by [RFC 8200](https://www.rfc-editor.org/rfc/rfc8200.html) 
{: .notice--info} 

These RFCs defines IPv6 packet format, different extension headers, order of processing and various other features. For deeper understanding of the same please follow the RFCs.

The commonly used extension headers and their usage is defind the following [document](https://www.cisco.com/en/US/technologies/tk648/tk872/technologies_white_paper0900aecd8054d37d.html). 

## IPv6 Extension Header Processing 

As per the standards defined, all networking devices must process Extension Header under basic forwarding conditions or in the context of advanced features such as Access Lists. It identifies the protocol requirements that must be observed. ([Reference](https://www.cisco.com/en/US/technologies/tk648/tk872/technologies_white_paper0900aecd8054d37d.html)). The Hop-by-Hop Extension Header is the only EH that must be fully processed by all network devices. The Hop-by-Hop EH is similar to the IPv4 options and must be the first in a chain of extension headers. Because the Hop-by-Hop EH must be fully processed, it is handled by the CPU and the IPv6 traffic that contains a Hop-by-Hop EH will go through the slow forwarding path. Network devices are not required to process any of the other IPv6 extension headers when simply forwarding the traffic. For this reason, IPv6 traffic with one or more EHs other than Hop-by-Hop can be forwarded in hardware.

## Security concerns with IPv6 Extension Headers

[RFC 9288](https://datatracker.ietf.org/doc/rfc9288/) documents security implications of IPv6 Extension Headers. It discusses the operational and interoperability implications of discarding packets based on the IPv6 Extension Headers and IPv6 options they contain. It also provides advice on the filtering of such IPv6 packets at transit routers for traffic not directed to them, for those cases where such filtering is deemed as necessary. Therefore it becomes necessary for the network devices to be capable of filtering those packets that may have Extension Headers and which may pose threat to the network.

## NCS5500/NCS5700 capabilities to filter IPv6 Traffic with extension headers
