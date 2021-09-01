---
published: true
date: '2021-08-31 10:01 +0530'
title: Access-List Enhancements on NCS5500 J2 based platforms
author: Tejas Lad
excerpt: This document will introduce the behaviour of ACL on J2 based platforms
position: top
tags:
  - iosxr
  - cisco
  - ACL
  - J2
  - data plane protection
  - access-list
  - NCS 5500
  - NCS 5700
---
{% include toc icon="table" title="Access-List Enhancements on NCS5500 J2 based platforms" %} 

## Introduction

In our previous articles, we introduced the ACL features for NCS500 and NCS5500 platforms based on Qumran-MX, Jericho and Jericho+ based chipsets. 

![Screenshot 2021-08-31 at 10.50.11 AM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 10.50.11 AM.png)

We discussed [ACL implementation](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/), [Hybrid ACL](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/), matching criterias like [Packet Length](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/), [IP Fragments](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/). We also discussed the other important features like [UDK](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/) and [UDF](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/). Finally we touched upon the concepts of [ABF](https://xrdocs.io/ncs5500/tutorials/acl-based-forwarding-and-object-tracking-for-ncs5xx-and-ncs55xx/) and [Chained ACL](https://xrdocs.io/ncs5500/tutorials/chained-acl-for-ncs55xx-and-ncs5xx/). In this artcile and the ones to follow this, we will explore the ACL enhancements on Jericho2 based platforms. 

![Screenshot 2021-08-31 at 10.54.58 AM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 10.54.58 AM.png)

There have been quite a few changes in implementation and support for ACLs on Jericho2 based platforms. Right from the permit stats availability to the programming of Ingress ACL on eTCAM. We also have introduced the support for more Default Keys for both ingress and egress ACLs. We no more need recyling of the IPv6 egress ACLs. We have the more support on the BVI interfaces compared to previous platforms. So let us start looking into each of them.

## Permit Stats 

Let us first start with the permit statistics. As we know, when it comes to NCS5500 and NCS500, we have limited hardware resources and we need to use them wisely if we need to accomodate different features together. In these platforms, by default ACL permit stats is not accounted for in the ingress direction due to resource sharing. We need to enable _hw-module profile stats acl-permit_, to allocate statistic entries to permit ACEs. But there is a drawback after enabling this profile. If acl-permit is configured, qos-enhanced or other options are disabled. But with J2 based platforms, the above permit stats CLI is no longer needed. By default we can allocate statistic entries.

Let us check this with an example. We have a NC57-18DD-SE in slot 3. We apply the ACL in the ingress direction and send traffic from IXIA.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show platform
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/CPU0          NC55-36X100G-A-SE          IOS XR RUN        NSHUT
0/0/NPU0          Slice                      UP                
0/0/NPU1          Slice                      UP                
0/0/NPU2          Slice                      UP                
0/0/NPU3          Slice                      UP                
<mark>0/3/CPU0          NC57-18DD-SE               IOS XR RUN        NSHUT</mark>

</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list permit-stats
 10 permit ipv4 25.1.7.0 0.0.0.255 any
 20 deny ipv4 26.1.7.0 0.0.0.255 any
 
RP/0/RP0/CPU0:5508-2-74142I-C#show running-config interface fourHundredGigE 0/3/0/21
interface FourHundredGigE0/3/0/21
 cdp
 ipv4 address 30.1.1.2 255.255.255.0
 <mark>ipv4 access-group permit-stats ingress</mark>
!
</code>
</pre>
</div>

### Verification

![Screenshot 2021-08-31 at 6.19.02 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 6.19.02 PM.png)

We have **not** configured _**hw-module profile stats acl-permit**_ 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show running-config | in hw-module 
hw-module profile qos hqos-enable
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:5508-2-74142I-C#show access-lists ipv4 permit-stats hardware ingress location 0/3/CPU0 
ipv4 access-list permit-stats
<mark> 10 permit ipv4 25.1.7.0 0.0.0.255 any (2290044 matches)</mark>
 20 deny ipv4 26.1.7.0 0.0.0.255 any
RP/0/RP0/CPU0:5508-2-74142I-C#
RP/0/RP0/CPU0:5508-2-74142I-C#
</code>
</pre>
</div>

The above output shows that Jericho2 based platforms, no longer need the hw-module profile for ingress stats. The stats are enabled by default. 

## Ingress ACL on External TCAM

Prior to IOS-XR release 7.2.1, traditional ingress IPv4/IPv6 ACLs were always programmed on the internal TCAM of a line card or fixed system be it a base or scale version. From IOS-XR 7.2.1, the programming of the ingress ACLs will be done on the external TCAM for the J2 based scale systems. Let us verify the same with an example. We have an access-list configured as below and applied in the ingress direction on interface of NC57-18DD-SE

```
ipv4 access-list permit-stats
 10 permit ipv4 25.1.7.0 0.0.0.255 any
 20 deny ipv4 26.1.7.0 0.0.0.255 any
 30 permit ipv4 host 50.1.1.1 any
 35 deny ipv4 62.6.69.128 0.0.0.15 any
 40 deny ipv4 any 62.80.66.128 0.0.0.15
 45 deny ipv4 62.80.66.128 0.0.0.15 any
 50 deny ipv4 any 62.134.38.0 0.0.0.127
 60 permit tcp any eq bgp host 1.2.3.1
 70 permit tcp any host 1.2.3.1 eq bgp
 80 deny ipv4 any host 1.2.3.1
 90 deny ipv4 any 212.21.217.0 0.0.0.255
 100 permit ipv4 any any
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface FourHundredGigE0/3/0/21
 cdp
 ipv4 address 30.1.1.2 255.255.255.0
 <mark>ipv4 access-group permit-stats ingress</mark>
!
</code>
</pre>
</div>

![Screenshot 2021-08-31 at 11.15.11 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 11.15.11 PM.png)

![Screenshot 2021-08-31 at 11.21.55 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 11.21.55 PM.png)

The above output shows that ACL programming has been done in the external TCAM. The interface belongs to NPU 1. It used the bank ID 15 and the database alloted for the ingress v4 ACL. It shows 15 entries per DB (12 ACEs plus 3 internal entries). 

### Summary of TCAM Usage

Let us summarise the TCAM usage for the ingress and egress ACLs w.r.t TCAMs used.


| System           | Traditional Ingress ACL | Ingress ACL with UDK/UDF | Egress ACL    | Hybrid Ingress ACL |
|------------------|-------------------------|--------------------------|---------------|--------------------|
| J2 with eTCAM    | External TCAM           | Internal TCAM            | Internal TCAM | External and Internal TCAM      |
| J+ with eTCAM    | Internal TCAM           | Internal TCAM            | Internal TCAM | External and Internal TCAM      |
| J with eTCAM     | Internal TCAM           | Internal TCAM            | Internal TCAM | External and Internal TCAM      |
| J2 without eTCAM | Internal TCAM           | Internal TCAM            | Internal TCAM | Not Supported      |
| J+ without eTCAM | Internal TCAM           | Internal TCAM            | Internal TCAM | Not Supported      |
| J without eTCAM  | Internal TCAM           | Internal TCAM            | Internal TCAM | Not Supported      |


Note: This is applicable for both J2 Native and Compatible Mode.
{: .notice--info}

### Advantage of programming ingress ACLs in external TCAM

From the above table, we can see that when using a fixed system or Line card with external TCAM, the traditional ingress ACLs are programmed on external TCAM. The main advantage is that the resources on the internal TCAM can now be used for other features and statistics.

## Ingress and Egress Default TCAM Keys

In NCS5500 and NCS500, we have the concept of default TCAM keys and user-defined TCAM keys- UDK. For details on the two different key types please [refer](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/). Due to enhance capabilities of J2 chipset we have made changes to the default TCAM key support for IPv4 and IPv6 both in ingress and egress directions. 

### IPV4 Ingress Default TCAM keys

| IPv4 Match Fields             | Support | Comment                                                                          |
|-------------------------------|---------|----------------------------------------------------------------------------------|
| Src Address                   | Yes     |                                                                                  |
| Dst Address                   | Yes     |                                                                                  |
| Source Port or ICMP Code/Type | Yes     |                                                                                  |
| Destination Port              | Yes     |                                                                                  |
| TOS/DSCP                      | Yes     |                                                                                  |
| Packet Length                 | Yes     | Part of the default TCAM key from IOS-XR 7.2.1. Previously allowed only with UDK |
| TCP Flags                     | Yes     |                                                                                  |
| Fragments                     | Yes     | Part of the default TCAM key from IOS-XR 7.2.1. Previously allowed only with UDK |
| Fragment Offset               | Yes     |                                                                                  |
| Protocol                      | Yes     |                                                                                  |

### IPV6 Ingress Default TCAM keys

| IPv6 Match Fields             | Support | Comment                                                                          |
|-------------------------------|---------|----------------------------------------------------------------------------------|
| Src Address                   | Yes     |                                                                                  |
| Dst Address                   | Yes     |                                                                                  |
| Source Port or ICMP Code/Type | Yes     |                                                                                  |
| Destination Port              | Yes     |                                                                                  |
| Traffic Class (DSCP)          | Yes     |                                                                                  |
| Packet Length                 | Yes     | Part of the default TCAM key from IOS-XR 7.2.1. Previously allowed only with UDK |
| TCP Flags                     | Yes     |                                                                                  |
| Next-header                   | Yes     |                                                                                  |


### IPV4 Egress Default TCAM keys

| IPv4 Match Fields             | Support | Comment                                                                          |
|-------------------------------|---------|----------------------------------------------------------------------------------|
| Src Address                   | Yes     |                                                                                  |
| Dst Address                   | Yes     |                                                                                  |
| Source Port or ICMP Code/Type | Yes     |                                                                                  |
| Destination Port              | Yes     |                                                                                  |
| TOS/DSCP                      | Yes     |                                                                                  |
| Packet Length                 | Yes     | Part of the default TCAM key from IOS-XR 7.4.1. Previously allowed only with UDK |
| TCP Flags                     | Yes     |                                                                                  |
| Fragments                     | Yes     |                                                                                  |
| Protocol                      | Yes     |                                                                                  |


### IPV6 Egress Default TCAM keys

| IPv6 Match Fields             | Support | Comment                                                                          |
|-------------------------------|---------|----------------------------------------------------------------------------------|
| Src Address                   | Yes     |                                                                                  |
| Dst Address                   | Yes     | Changed to 96 bits only                                                          |
| Source Port or ICMP Code/Type | Yes     |                                                                                  |
| Destination Port              | Yes     |                                                                                  |
| Traffic Class/DSCP            | Yes     | Part of the default TCAM key from IOS-XR 7.4.1. Previously allowed only with UDK |
| Packet Length                 | Yes     | Part of the default TCAM key from IOS-XR 7.4.1. Previously allowed only with UDK |
| TCP Flags                     | Yes     | Part of the default TCAM key from IOS-XR 7.4.1. Previously allowed only with UDK |
| Fragments                     | Yes     | Part of the default TCAM key from IOS-XR 7.4.1. Previously allowed only with UDK |
| Next Header                   | Yes     |                                                                                  |

Note: This is applicable for both J2 Native and Compatible Mode.
{: .notice--info}

## Scale 

As of IOS-XR 7.4.1, the scale on the J2 based platforms will be same as previous generations. There is a plan to increase the scale in future releases. Stay tuned !!!

## Thank You

Special thanks to Shruthi (shrucs@cisco.com) for her valuable inputs during the article.

## Summary

Hope this article was helpful. We covered the new enhancements/support and the programming difference in the new platforms. In the upcoming articles we will cover the IPv6 egress ACL's, ACL for IPv6 EH support, ACL support on BVI interface. We will also have a separate article on how to increase the scale. Stay tuned for the same !!!



