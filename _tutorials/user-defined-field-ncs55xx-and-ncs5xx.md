---
published: true
date: '2020-09-02 15:39 +0530'
title: User Defined Field NCS55xx and NCS5xx
author: Tejas Lad
position: hidden
excerpt: This documents aims to deep dive into UDF ACL for NCS55xx and NCS5xx ACL's
tags:
  - NCS5500
  - NCS500
  - NCS55xx
  - UDF
  - ACL
---
{% include toc %}

## Introduction

In the previous [tech-note](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/ "tech-note"), we had discussed the concept of User Defined Keys - UDK. We also compared the UDK with Default TCAM keys and the ways to optimize the memory usage. In this tech-note, we will deep dive into advanced filtering capabilities of NCS5xx and NCS55xx with User Defined Fields - UDF. 

## Overview

![]({{site.baseurl}}/images/Screenshot%202020-08-27%20at%203.33.28%20PM.png)

In most cases, matching criterias are statically defined. Users do not have control over the bits in the packet, they want to use for matching. However, in some cases, the user may desire to match on a set of bits in the packet which is not associated with a specific, pre-defined header field. This means we need a way to classify on new fields in addition to the existing fields. But there is a catch here. It is not straight forward to add new fields in addition to existing fields, as we have fixed key length in the TCAM. It may not be always possible to have room for all the new fields. Other limitations with standard ACL's is, it does not provide granularity when dealing with tunneled traffic. Due to these criterias, hardware must be capable of matching traffic based on user defined fields.  

User Defined Field's or UDF can be considered as an inspection of a packet based on offset values. An ACL can be defined with UDF matching capabilities to give granularity and flexibility when identifying traffic patterns. It is often used for deeper packet analysis. Typical use cases includes, finding out patterns inside inner header when packets are tunneled. Another use case is identifying traffic to mirror monitor sessions for analysis. Any offset value within 128 bytes in a packet can be matched. We will see different use cases in details in later section.

## UDF Feature Support 

  - UDF feature is supported only in the Ingress direction. It is not supported on the Egress direction.
  - It is supported on NCS540, NCS560 and NCS5500 (J/J+/J2).
  - The ingress PMF supports packet inspection of the first 128 bytes of a packet. 
  - The offset support is limited to 63 bytes from the beginning of a header.
  - It is supported only on Layer3 interfaces.
  - It is not suported on Layer2 interfaces.
  - As of 7.2.1, only 7 UDF's are supported. Prior to 7.2.1, 8 UDF's were supported.
  - No support for inner L2 header. It is only supported for outer header.
  - Mask Length is 4 bytes and Length 0 is not supported.
  - UDF name can be maximum of 16 characters.
  - UDF is not supported in default TCAM keys. It must be used along with User-Defined TCAM Keys.
  - UDF is supported for both IPv4 and IPv6.
  
 
## UDF Definition and Configuration

  - The user needs to define the name of the UDF, the header (inner/outer), offset and length of data to extract. 
  
  ![]({{site.baseurl}}/images/Screenshot%202020-08-27%20at%2011.35.14%20PM.png)
  
  - Then we need to define a UDK referencing the UDF

<div class="highlighter-rouge">
<pre class="highlight">
<code>
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port <mark>udf1 udf_test</mark>
</code>
</pre>
</div>

The hw-module verifies that the user has requested a valid tcam key. For IPv4 this will be either a 160 or 320 bit key. For IPv6 it will be a 320 bit key. 

  - Then it needs to be defined in the ACE 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list udf_acl
<mark>10 deny ipv4 any any udf udf_test 0x4567 0xffff</mark>
</code>
</pre>
</div>

## UDF Use cases

### Matching layer 2 entities on a Layer 3 interface

- Consider the below toplogy

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%201.33.33%20PM.png)
