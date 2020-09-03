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

We have a layer 3 connectivity end to end and host 60.1.1.2 wants to reach 70.1.1.2. In a normal sceanrio, we wont we able to match the VLAN ID on a layer 3 interface. For doing that we would need to define a Layer 2 ACL. But that wont be allowed to configure on a Layer 3 interface. Now what if a network administrator wants to match or filter a packet on his desired field (in this case consider VLAN id). If the platform does not have the capability to filter on deep packet analysis, network administrator will not be able to achieve that. Thanks to UDF capability of NCS55xx and NCS5xx, we can filter packets on user defined fields.

**Configuring UDF**

Consider the packet below. We are sending the packet with source address 60.1.1.2 and destination address 70.1.1.2 with an encapsulation of 10.

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%201.42.58%20PM.png)

If we want to filter the packet on the basis of encapsulation 10 we need to define the below UDF globally.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>udf udf_vlan header outer l2 offset 14 length 2</mark>
</code>
</pre>
</div>

**Referring the UDF to the ACL and applying it on the interface**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list UDF_VLAN
<mark>10 deny ipv4 any any udf udf_vlan 0xa 0xffff</mark>
 20 permit ipv4 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface TenGigE0/0/0/0.10
 description using it for ACL testing
 ipv4 address 60.1.1.1 255.255.255.0
 ipv6 address 60::1/64
 load-interval 30
 encapsulation dot1q 10
<mark> ipv4 access-group UDF_VLAN ingress</mark>
</code>
</pre>
</div>


**Verifying UDF**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location 0/0/CPU0 
Mon Aug 31 08:27:27.590 UTC
Interface : TenGigE0/0/0/0.10 
<mark>    Input  ACL : Common-ACL : N/A  ACL : UDF_VLAN</mark>
    Output ACL : N/A
</code>
</pre>
</div>    

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Mon Aug 31 08:28:34.944 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1898     91      36   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1898     19      45   INGRESS_RX_ISIS
0    0      160b   pmf-0       1898     23      54   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1898     15      56   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1898     2       60   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1992     4       47   INGRESS_ACL_L3_IPV4</mark>
</code>
</pre>
</div> 


``` 
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_VLAN hardware ingress verify location 0/0/CPU0 
Mon Aug 31 08:27:49.483 UTC

Verifying TCAM entries for UDF_VLAN
Please wait...                                   
                                                 
 
 
    INTF    NPU lookup  ACL # intf Total  compression Total   result failed(Entry) TCAM entries 
                type    ID  shared ACES   prefix-type Entries        ACE SEQ #     verified
 ---------- --- ------- --- ------ ------ ----------- ------- ------ ------------- ------------
 
TenGigE0_0_0_0.10 (ifhandle: 0x41b8)

              0 IPV4      7      1      2 NONE              3 passed                          3

 ```


![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%202.33.46%20PM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_VLAN hardware ingress location 0/0/CPU0 
Mon Aug 31 09:05:35.701 UTC
ipv4 access-list UDF_VLAN
<mark> 10 deny ipv4 any any (43041269 matches)</mark>
 20 permit ipv4 any any
</code>
</pre>
</div> 


We can see the end to end traffic is getting dropped.

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%202.37.10%20PM.png)


