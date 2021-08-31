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

We discussed [ACL implementation](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/), [Hybrid ACL](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part2-hybrid-acl/), matching criterias like [Packet Length](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/), [IP Fragments](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/). We also discussed the other important features like [UDK](https://xrdocs.io/ncs5500/tutorials/user-defined-key-udk-for-ncs55xx-and-ncs5xx/) and [UDF](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/). Finally we touched upon the concepts of [ABF](https://xrdocs.io/ncs5500/tutorials/acl-based-forwarding-and-object-tracking-for-ncs5xx-and-ncs55xx/) and [Chained ACL](https://xrdocs.io/ncs5500/tutorials/chained-acl-for-ncs55xx-and-ncs5xx/). In this artcile and the ones to follow this we will explore the ACL enhancements on Jericho2 based platforms. 

![Screenshot 2021-08-31 at 10.54.58 AM.png]({{site.baseurl}}/images/Screenshot 2021-08-31 at 10.54.58 AM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark></mark>

</code>
</pre>
</div>