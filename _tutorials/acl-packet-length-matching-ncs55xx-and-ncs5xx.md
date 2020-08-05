---
published: true
date: '2020-08-04 11:16 +0530'
title: ACL Packet Length Matching - NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document describes the implementation of ACL packet length matching on
  NCS55xx and NCS5xx
position: top
tags:
  - cisco
  - NCS5500
  - NCS540
  - NCS560
  - ACL
  - NCS 5500
---
{% include toc %}

## Introduction

Access Control Lists have been implemented for a long time now and has been integral part of data plane security for almost every organization.

![Screenshot 2020-08-04 at 11.03.21 AM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 11.03.21 AM.png)

Though everyone would be aware of what an access control list is, I would like to brush up some basics before deepdiving into complex functionalities.

ACL’s can be considered as an ordered list of conditions used to test the network traffic that traverses through the router interfaces. On the basis of the defined lists, the router decides which packets to accept and which to drop. ACL’s help in managing the traffic and secure the access to and from the network.

ACL’s can make permit/deny decisions based on source/destination address, source/destination ports, L3 protocols, L4 port numbers and many others.

## Overview

Security ACL's introduction, feature support and statistics is covered at high level in the [Link](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Link")

In this document, we will deepdive how the NCS55xx and NCS5xx program the packet length in the TCAM and use it to filter the packets. The main use case of this matching criteria is to identify malicious packet ranges entering the network and denying them.  

ACL’s on NCS55xx and NCS5xx, uses the Programmable Mapping and Filtering (PMF) functionality and TCAM (internal/exertnal) in both the Ingress Receive Packet Processing (IRPP) blocks and Egress Receive Packet Processing (ERPP) blocks. The line cards in these platforms are based on the Broadcom family of chipsets. These chipsets uses a pipeline architecture which has dedicated hardware blocks for performing various functions.

ACL’s contains one or more ACEs which are used to match packets and perform an action on those packets. Typical action of the ACE is to either Permit or Deny. The TCAM is programmed with tables, may be like a database on which the match criteria and action criteria are performed.

In hardware, we have databases that are unique to each feature. For ACL, we have further defined unique databases based on these fields:

  - Protocol (IPv4, IPv6, L2)
  - Direction (ingress/egress)
  - Compression (uncompressed/compressed)
  - Default TCAM key or user-defined (UDK) TCAM key

## Supported ACL Match Criteria 

  - Packet Length 
  - IP Fragmentation
  - Source and Destination Port
  - User Defined Keys - UDK
  - User Defined Field - UDF

Note: We will have dedicated posts for explaining each matching criterias.
{: .notice--info}

## Header Definition - IPv4

![Screenshot 2020-08-05 at 10.31.55 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 10.31.55 PM.png)


In NCS55xx and NCS5xx, when we configure an ACE through CLI, the total IP packet includes only IPv4 header. As per the above figure, only IP payload is taken into the consideration when you define the packet length in an ACE. It does not include any L2 headers, including ethernet/vlan. Therefore, when matching the packets on the router, the layer 2 headers needs to be taken into consideration and the packet length value should be configured accordingly. We will see this with an example in later section.
  

### Packet matching criterias

We have different criterias for the matching

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24(config-ipv4-acl)#40 permit ipv4 any any packet-length ?
   <mark>eq     Match only packets with a given value
  gt     Match only packet with a greater value
  lt     Match only packets with a lower value
  neq    Match only packets not on a given value
  range  Match only packets in the range of value</mark>
RP/0/RP0/CPU0:N55-24(config-ipv4-acl)#40 permit ipv4 any any packet-length 
</code>
</pre>
</div>

### ACL configuration

Lets us configure a simple ACL for matching packet length and attaching it to the interface

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 test-acl-v4-pkt-length 
Thu Jul 23 06:46:11.884 UTC
ipv4 access-list test-acl-v4-pkt-length
 10 permit ipv4 any any packet-length eq 800
 20 permit ipv4 any any packet-length eq 1000
 30 permit ipv4 any any packet-length eq 1500
RP/0/RP0/CPU0:N55-24#
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show running-config interface tenGigE 0/0/0/0.10
Thu Jul 23 06:46:34.378 UTC
interface TenGigE0/0/0/0.10
 description using it for ACL testing
 ipv4 address 60.1.1.1 255.255.255.0
 load-interval 30
 encapsulation dot1q 10
 <mark>ipv4 access-group test-acl-v4-pkt-length ingress</mark>
!
</code>
</pre>
</div>


Note: After IOS-XR release, 6.5.2 and later the packet length is not supported by default TCAM key and we need to configure a UDK to have it in the key. 
{: .notice--info}

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit port-range
```


### ACL Verification

![Screenshot 2020-08-05 at 2.22.26 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 2.22.26 PM.png)

![Screenshot 2020-08-05 at 2.23.29 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 2.23.29 PM.png)

Other show commands are extensively covered in the [Link](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Link")

### Hardware or TCAM programming of the ACL

![Screenshot 2020-08-05 at 2.44.28 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 2.44.28 PM.png)

![Screenshot 2020-08-05 at 2.46.09 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 2.46.09 PM.png)

The above 2 output shows us that the IPv4 L3 ACL database is programmed and a Database ID is created for it. NPU details is extracted, dedicated to interface where the ACL is applied. We can see the bank_ID with the entry size as 320 bits. For more information on memory banks please [Refer](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Refer"). This data/values will help us in understanding the values configured in the hardware.
  
![Screenshot 2020-08-04 at 12.24.17 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 12.24.17 PM.png)

The above output shows the TCAM programming of the packet length configured in Hexadecimal

| Hexadecimal | Decimal |
|-------------|---------|
| 320         | 800     |
| 3E8         | 1000    |
| 5DC         | 1500    |



### Traffic Tests

  - Below is the snapshot of the traffic stream used. 
  - It has a packet length of 822 bytes. (800 bytes plus 18 bytes of the Ethernet header + 4 bytes VLAN header)
  

![Packetview.png]({{site.baseurl}}/images/Packetview.png)

![Traffic Flow.png]({{site.baseurl}}/images/Traffic Flow.png)


The traffic is matching the first ACE with packet length of 800 bytes. As mentioned earlier, that the TCAM doesnt consider L2 headers. So the traffic stream has to be sent accordingly. In real production network, the ACE has to be configured accordingly so we can permit or deny legitimate packets. This can be done by configuring range command. It will be explained in the later section


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 test-acl-v4-pkt-length hardware ingress location 0/0/CPU0
Thu Jul 23 08:40:42.081 UTC
ipv4 access-list test-acl-v4-pkt-length
 <mark>10 permit ipv4 any any packet-length eq 800 (4124541 matches)</mark>
 20 permit ipv4 any any packet-length eq 1000
 30 permit ipv4 any any packet-length eq 1500
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>


Note: Permit ACL stats are not enabled by default. We need to configure the below hw-module profile to enable the same 
{: .notice--info}

```
hw-module profile stats acl-permit

```

### Changing the frame size 

Modifying the packet length to 800: traffic drops as the packet does not match any ACE. For the traffic to match we need to configure an ACE with packet length 800-22 = 778 bytes, due to the reason stated above.

![TrafficDrop.png]({{site.baseurl}}/images/TrafficDrop.png)

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 test-acl-v4-pkt-length hardware ingress location 0/0/CPU0
Thu Jul 23 08:49:28.720 UTC
ipv4 access-list test-acl-v4-pkt-length
 10 permit ipv4 any any packet-length eq 800
 20 permit ipv4 any any packet-length eq 1000
 30 permit ipv4 any any packet-length eq 1500
RP/0/RP0/CPU0:N55-24#

```

## Packet Length range 

For scenarios, where we are not sure on the absolute packet length, we have the option to configure range. In the below ACL, we have confgured 2 ACE's. Sequence 10 is a range and sequence 20 is an absolute value.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list test-acl-v4-pkt-length
 <mark>10 permit ipv4 any any packet-length range 800 1000</mark>
 20 permit ipv4 any any packet-length eq 1500
</code>
</pre>
</div>

![Screenshot 2020-08-05 at 3.53.42 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 3.53.42 PM.png)

We can see that with range command only 8 (7+ 1 internal usage) entries are consumed in the TCAM. If we configure different values that would utilize one entry each.

![Screenshot 2020-08-05 at 4.05.30 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 4.05.30 PM.png)


### Hardware Programming of the range

![Screenshot 2020-08-04 at 1.47.37 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-04 at 1.47.37 PM.png)


As per the above ACL, TCAM programmed values

| Hexadecimal | Decimal    | Value                     |
|-------------|------------|---------------------------|
| 3E8         | 1000       | Absolute Value programmed |
| 3E0/FFF8    | 992 to 999 | Range programmed          |
| 3C0/FFE0    | 960 to 991 | Range programmed          |
| 380/FFC0    | 896 to 959 | Range programmed          |
| 340/FFC0    | 832 to 895 | Range programmed          |
| 320/FFE0    | 800 to 831 | Range programmed          |
| 5DC         | 1500       | Absolute Value programmed |


When configuring the range command the algorithm takes into account the mask as well the value. For example, 3E0/FFF8 has 3E0 as value and FFF8 is mask. Accordingly it programs all the values in batches of entries for the given range into the TCAM. Let us understand, how the range is configured in the TCAM w.r.t value/mask pair and how to interpret it.

```
3E0/FFF8
3E0 -- Value
FFF8 -- Mask
FFF8 in binary is 1111111111111000
The last 3 bits are 0 
So we can program 2^3 = 8 values. Which means 3E0 to 3E7
3E0 = 992
3E7 = 999

```
  - Hence, the key here is the bits in the mask with 0's, according to which the values are programmed in the TCAM



## Header Definition - IPv6

For IPv6 the "payload length" field in the packet does not include the IPv6 headers. It only covers the payload length (the data following this headers).The IPv6 header is assumed to be 40 bytes; so the configured ACE's packet length is reduced by 40 bytes and this value is configured into the TCAM as the match criteria for the IPv6 header. For example, if the ACE is configured for the packet length of 200, the TCAM will configure it as 160 

The IPv6 header is really not a fixed size, because there can be one or more extension headers. Currently, the hardware does not take this into consideration and simply assumes the IPv6 header is a fixed 40 bytes.
We can similarly apply a IPv6 ACL and check the programming using the same commands._We will dedicate the a separate post for IPv6 Extension Header_


## Optimizing Memory usage

Sometimes in real production networks, we are not sure what packet length we will receive on the interface. Whether it have a VLAN header or not, or what will be the size of the packet. In this scenarios, it is recommended to use range option. This will help in optimizing resource utilization with lesser TCAM entries. For example, if we want to permit packets only with length 800 to 810 bytes while denying others.
This will consume 13 entries in the TCAM
  
![Screenshot 2020-08-05 at 4.35.02 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 4.35.02 PM.png)

  - Instead of using the individual ace's if we use range option, we will use only 5 entries
  
![Screenshot 2020-08-05 at 4.40.14 PM.png]({{site.baseurl}}/images/Screenshot 2020-08-05 at 4.40.14 PM.png)


## References

- [Security ACL's on NCS5500](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "Security ACL's on NCS5500")
- [CCO documentation](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "CCO documentation")

## Summary

Hope this document helps to understand the matching criteria on the basis of packet length. This can be particularly useful in mitigating packets with sizes which are known for malicious behaviour. Those can be detected and prevented from causing data plane security issues.

We also saw how to utilize the internal tcam resources optimally by using range command. This is particularly useful when we have many ACE's in traditional ACL's. Configuring higher or lower packet length doesnt cause the TCAM entries to increase.

Stay tuned for the next matching criteria and its interpretation at the hardware level.
