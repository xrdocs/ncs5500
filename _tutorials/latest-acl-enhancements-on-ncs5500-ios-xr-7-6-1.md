---
published: true
date: '2022-04-18 13:10 +0530'
title: Latest ACL Enhancements on NCS5500 - IOS-XR 7.6.1
author: Tejas Lad
excerpt: >-
  This document will introduce the behaviour new enhancements w.r.t ACL feature
  on NCS5500
tags:
  - iosxr
  - cisco
  - ACL
  - NCS5500
  - access-list
  - access control list
  - data plane
position: hidden
---
{% include toc icon="table" title="Latest ACL Enhancements on NCS5500 - IOS-XR 7.6.1" %} 
## Introduction

In our [previous article](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/), we discussed the ACL enhancements on the newer generation NCS5500 plaforms. In this article we will discuss some more enhancements w.r.t ACLs, which we have done in IOS-XR 7.6.1

## IOS-XR 7.6.1 Enhancements

In IOS-XR 7.6.1, we have brought in a number of new enhancements when it comes to ACLs. 
 - Increased Ingress ACLs
 - ACL Chaining with ACL Based Forwarding (ABF)
 - Enable Ingress Interface Logging on ACE
 - ACL-Based Policing

We will discuss about each enhancement in details

## Increased Ingress ACLs


In earlier releases, we could configure maximum upto 127 different traditional ingress ACLs and 255 different hybrid ingress ACLs in shared ACL mode per line card. From IOS-XR 7.6.1 we can now configure an increased number of either traditional (non-compression) or hybrid (compression) ingress ACLs in shared ACL mode, as listed below:

 - A maximum of 512 different traditional ingress ACLs per line card.
 - A maximum of 1000 different hybrid ingress ACLs per line card.

Increased ACLs provide you with enhanced traffic filtering capabilities to control how traffic packets move through the network and restrict the access of users and devices to the network. For further deepdive please follow the [link](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/76x/b-ip-addresses-cg-ncs5500-76x/m-implementing-access-lists-prefix-lists-ncs5500.html#id_63255). 

## ACL Chaining with ACL Based Forwarding (ABF)

Prior to IOS-XR 7.6.1, ABF and ACL chaining with Common ACL were mutually exclusive features. From 7.6.1 onwards, we can enable ABF in conjunction with ACL chaining in Common ACL. With this feature, the router can inspect and forward the packets based on the ABF rule in Common ACL. In earlier releases. For further deepdive, please follow the links for [Chained ACL](https://xrdocs.io/ncs5500/tutorials/chained-acl-for-ncs55xx-and-ncs5xx/) and [ABF](https://xrdocs.io/ncs5500/tutorials/acl-based-forwarding-and-object-tracking-for-ncs5xx-and-ncs55xx/).

## Enable Ingress Interface Logging on ACE

NCS5500 product family already supports logging messages about packets permitted or denied by IP access list. That is, any packet that matches the access list causes an informational logging message about the packet. The level of messages logged to the console is controlled by the logging console command in global configuration mode. The first packet that triggers the access list causes an immediate logging message, and subsequent packets are collected over 5-minute intervals before they are displayed or logged. ([Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/76x/b-ip-addresses-cg-ncs5500-76x/m-implementing-access-lists-prefix-lists-ncs5500.html#con_1063757)). Let us verify the behaviour on the router.

We have a router with ACL configured with logging enabled.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57#show access-lists ipv4 acl_log 
ipv4 access-list acl_log
 10 permit icmp host 172.16.0.57 host 172.16.0.53 <mark>log</mark>
 20 permit icmp host 172.16.0.53 host 172.16.0.57 <mark>log</mark>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57#
</code>
</pre>
</div>

When we hit a ACE, we see the below in the syslogs 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57#RP/0/RP0/CPU0:Apr 19 09:00:41.213 UTC: ipv4_acl_mgr[162]: %ACL-IPV4_ACL-6-IPACCESSLOGDP : <mark>access-list acl_log (20) permit icmp 172.16.0.53 -> 172.16.0.57 (8/0), 4406 packets</mark> 
</code>
</pre>
</div>

If we do not specify the log keyword we would not have received this particular syslogs in the console. This gives us better readability of the syslogs as compared to no logging option.

You can control the number of packets that, when they match an access list (and are permitted or denied), cause the system to generate a log message. You might do this to receive log messages more frequently than at 5-minute intervals. The below command helps to achieve the same

```
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#ipv4 access-list log-update threshold ?
  <1-2147483647>  Log update threshold (number of hits)
RP/0/RP0/CPU0:N57B1-1-Vega-II5-57(config)#ipv4 access-list log-update threshold
```

Note: The similar is applicable for IPv6 Access-list as well.
{: .notice--info}

From IOS-XR 7.6.1, we have enhanced the logging feature for the ACL to give more readability to the users. We have introduced the keyword **log-input**. This is an optional keyowrd and it provides the same functionality as the log keyword, as described above, except that the log-message also includes the ingress interface on which the router receives the packet. The router supports this feature for both IPv4 and IPv6 ingress ACLs on:
 - Physical Interfaces
 - Sub-interfaces and 
 - Bridged-virtual interfaces (BVI)
 - Bundle Interfaces
 
 Whenever a permit/deny ACE is hit we get a syslog which also mentions the interface details 
 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
 RP/0/RP0/CPU0:N57B1-1-Vega-II5-57#RP/0/RP0/CPU0:Apr 19 09:32:29.700 UTC: ipv4_acl_mgr[162]: %ACL-IPV4_ACL-6-IPACCESSLOGDP : access-list acl_log (20) permit icmp 172.16.0.53 <mark>HundredGigE0/0/0/8-> 172.16.0.57 (8/0), 1 packet</mark> 
</code>
</pre>
</div>

