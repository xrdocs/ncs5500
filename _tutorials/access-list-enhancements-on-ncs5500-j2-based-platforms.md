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

## Permit Stats 

Let us first start with the permit statistics. As we know, in the platforms with J/J+/Q-MX we have limited hardware resources and we need to use them wisely if we need to accomodate different features together. In these platforms, by default ACL permit stats is not accounted for in ingress direction due to resource sharing. We need to enable _hw-module profile stats acl-permit_, to allocate statistic entries to permit ACEs. But there is a drawback after enabling this profile. If acl-permit is configured, qos-enhanced or other options are disabled. But with J2 based platforms, the above permit stats CLI is no longer needed. By default we can allocate statistic entries.

Let us check this with an example. We have a Jericho2 based platform in slot 3. We apply the ACL in the ingress direction and permit the end to end traffic from IXIA

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

We have not configured the _hw-module profile stats acl-permit_ 

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

So we have verified that in Jericho2 based platforms we no longer need to configure the hw-module profile for ingress stats. They are enabled by default. We can automatically see the permit and deny hits for the applied ACL's

