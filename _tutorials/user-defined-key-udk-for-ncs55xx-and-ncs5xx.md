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
position: top
---
{% include toc %}

## Introduction

In the previous technotes, ([ACL Packet Length Match](https://xrdocs.io/ncs5500/tutorials/acl-packet-length-matching-ncs55xx-and-ncs5xx/ "ACL Packet Length Match"), [ACL Fragment Match](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/ "ACL Fragment Match")) we have used the term User Defined Key - UDK many times and also saw it was compulsory to configure it for certain match criteria. In this technote, we will deep dive into the UDK concept and explore in details regarding the feature support.

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


Below table shows the frequently used qualifiers for IPv4 and IPv6. If the default TCAM key is set as Enabled, then the Qualifier field is enabled by default. If the default TCAM key is set as Disabled, then Qualifier field must use UDK. ([refer](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "refer"))

| Parameter            | IPv4 Default Key                  | IPv6 Default Key |
|----------------------|-----------------------------------|------------------|
| Source Address       | Enabled                           | Enabled          |
| Destination Address  | Enabled                           | Enabled          |
| Source Port          | Enabled                           | Enabled          |
| Destination Port     | Enabled                           | Enabled          |
| Port Range           | Enabled                           | Not Supported    |
| Protocol/Next Header | Enabled                           | Enabled          |
| Fragment bit         | Enabled (fragment-type needs UDK) | Not Supported    |
| Packet length        | Disabled                          | Disabled         |
| Precedence/DSCP      | Disabled                          | Enabled          |
| TCP Flags            | Enabled                           | Enabled          |
| TTL Match            | Disabled                          | Disabled         |
| Interface-based      | Disabled                          | Disabled         |
| UDF 1-7              | Disabled                          | Disabled         |
| ACL ID               | Enabled                           | Enabled          |


Note: This table is applicable across portfolio and also holds true for system with external tcam as well.
{: .notice--info}


## Feature Details

Let us explore the UDK support in details.


### UDK Definition


**Defining IPv4 UDK ACL Format**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-24(config)#hw-module profile tcam format access-list ipv4 ?</mark>
  common-acl       enable common-acl, 1 bit qualifier
  dst-addr         destination address, 32 bit qualifier
  dst-port         destination L4 Port, 16 bit qualifier
  enable-capture   Enable ACL based mirroring (Included by default)
  enable-set-ttl   Enable Setting TTL field (Included by default)
  frag-bit         fragment-bit, 1 bit qualifier
  interface-based  Enable non-shared interface based ACL
  location         Location of format access-list ipv4 config
  packet-length    packet length, 16 bit qualifier
  port-range       ipv4 port range qualifier, 24 bit qualifier
  precedence       precedence/dscp, 8 bit qualifier
  proto            protocol type, 8 bit qualifier
  src-addr         source address, 32 bit qualifier
  src-port         source L4 port, 16 bit qualifier
  tcp-flags        tcp-flags, 6 bit qualifier
  ttl-match        Enable matching on TTL field
  udf1             user defined filter
  udf2             user defined filter
  udf3             user defined filter
  udf4             user defined filter
  udf5             user defined filter
  udf6             user defined filter
  udf7             user defined filter
  udf8             user defined filter
</code>
</pre>
</div>


**Defining IPv6 UDK ACL Format**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-24(config)#hw-module profile tcam format access-list ipv6 ?</mark>
  common-acl       enable common-acl, 1 bit qualifier
  dst-addr         destination address, 128 bit qualifier
  dst-port         destination L4 Port, 16 bit qualifier
  enable-capture   Enable ACL based mirroring (Included by default)
  enable-set-ttl   Enable Setting TTL field (Included by default)
  interface-based  Enable non-shared interface based ACL
  location         Location of format access-list ipv6 config
  next-hdr         next header, 8 bit qualifier (manditory field)
  payload-length   payload length, 16 bit qualifier
  src-addr         source address, 128 bit qualifier
  src-port         source L4 Port, 16 bit qualifier (manditory field)
  tcp-flags        tcp-flags, 8 bit qualifier
  traffic-class    Traffic Class, 8 bit qualifier
  ttl-match        Enable matching on TTL field
  udf1             user defined filter
  udf2             user defined filter
  udf3             user defined filter
  udf4             user defined filter
  udf5             user defined filter
  udf6             user defined filter
  udf7             user defined filter
  udf8             user defined filter
</code>
</pre>
</div> 

**Example hw-module profile**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port packet-length frag-bit precedence port-range
hw-module profile tcam format access-list ipv6 src-addr src-port dst-addr dst-port next-hdr payload-length</mark>
</code>
</pre>
</div>

### UDK TCAM Size

Let us configure an IPv4 ACL as below 

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 test-acl-v4-pkt-length 
Sun Aug 16 06:24:34.093 UTC
ipv4 access-list test-acl-v4-pkt-length
 10 permit ipv4 any any packet-length range 800 831
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show running-config int tenGigE 0/0/0/0.10
Sun Aug 16 06:23:44.599 UTC
interface TenGigE0/0/0/0.10
 description using it for ACL testing
 ipv4 address 60.1.1.1 255.255.255.0
 ipv6 address 60::1/64
 load-interval 30
 encapsulation dot1q 10
 <mark>ipv4 access-group test-acl-v4-pkt-length ingress</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 06:26:57.373 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1902     97      30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1902     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1902     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1902     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1902     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
0    1      160b   pmf-0       1996     52      49   INGRESS_QOS_L2
0    2      160b   egress_acl  2031     17      17   EGRESS_QOS_MAP
0    3      160b   Free        2048     0       0    
0    4\5    320b   pmf-0       1999     27      31   INGRESS_LPTS_IPV6
<mark>0    4\5    320b   pmf-0       1999     3       39   INGRESS_ACL_L3_IPV4</mark>
0    4\5    320b   pmf-0       1999     19      47   INGRESS_QOS_IPV6
</code>
</pre>
</div>

Above output shows a ingress ACL in the TCAM occupying the key space of 320 bits.

Let us modify the hw-module profile format.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv4 src-addr dst-addr packet-length</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24(config)#interface tenGigE 0/0/0/0.10 
<mark>RP/0/RP0/CPU0:N55-24(config-subif)#ipv4 access-group test-acl-v4-pkt-length ingress</mark> 
RP/0/RP0/CPU0:N55-24(config-subif)#commit 
RP/0/RP0/CPU0:N55-24(config-subif)#end
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 06:54:01.114 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1902     97      30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1902     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1902     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1902     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1902     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1993     3       39   INGRESS_ACL_L3_IPV4</mark> 
0    1      160b   pmf-0       1993     52      49   INGRESS_QOS_L2
0    2      160b   egress_acl  2031     17      17   EGRESS_QOS_MAP
0    3      160b   Free        2048     0       0    
0    4\5    320b   pmf-0       2002     27      31   INGRESS_LPTS_IPV6
0    4\5    320b   pmf-0       2002     19      47   INGRESS_QOS_IPV6
</code>
</pre>
</div>

We can see after modifying the profile with a fewer keys the same ACL is occupying only 160 bits in the TCAM. This way users can define keys which can help optimize the TCAM resources.

Note: Changing of hw-module profile format will require reload of the router or line card depending on fixed or modular chassis.
{: .notice--info}


Let us see an example of IPv6 ACL and TCAM entry

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv6 src-addr src-port dst-addr next-hdr</mark>
</code>
</pre>
</div>

```
ipv6 access-list IPv6_ingress
 10 permit ipv6 any any

```
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24(config)#interface tenGigE 0/0/0/0.10  
<mark>RP/0/RP0/CPU0:N55-24(config-subif)#ipv6 access-group IPv6_ingress ingress</mark> 
RP/0/RP0/CPU0:N55-24(config-subif)#commit 
RP/0/RP0/CPU0:N55-24(config-subif)#end
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 08:36:05.722 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1902     97      30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1902     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1902     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1902     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1902     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
0    1      160b   pmf-0       1996     52      49   INGRESS_QOS_L2
0    2      160b   egress_acl  2031     17      17   EGRESS_QOS_MAP
0    3      160b   Free        2048     0       0    
0    4\5    320b   pmf-0       2002     27      31   INGRESS_LPTS_IPV6
0    4\5    320b   pmf-0       2002     19      47   INGRESS_QOS_IPV6
<mark>0    6\7    320b   pmf-0       2043     5       40   INGRESS_ACL_L3_IPV6</mark>
</code>
</pre>
</div>

As mentioned above, IPv6 ACL occupies 320 bits in the TCAM. We will see in later section how the size are calculated and are dependent on the configured UDK.

### UDK and Default-Key : Preference ? 

Consider below 2 simple IPv4 and IPv6 ACL's

```
ipv4 access-list test-ipv4
 10 permit ipv4 any any

ipv6 access-list IPv6_ingress
 10 permit ipv6 any any
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-24(config)#interface tenGigE 0/0/0/0.10
RP/0/RP0/CPU0:N55-24(config-subif)#ipv4 access-group test-ipv4 ingress 
RP/0/RP0/CPU0:N55-24(config-subif)#ipv6 access-group IPv6_ingress ingress</mark> 
RP/0/RP0/CPU0:N55-24(config-subif)#commit 
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 08:59:10.118 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1902     97      30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1902     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1902     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1902     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1902     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1993     3       39   INGRESS_ACL_L3_IPV4</mark>
0    1      160b   pmf-0       1993     52      49   INGRESS_QOS_L2
0    2      160b   egress_acl  2031     17      17   EGRESS_QOS_MAP
0    3      160b   Free        2048     0       0    
0    4\5    320b   pmf-0       2002     27      31   INGRESS_LPTS_IPV6
0    4\5    320b   pmf-0       2002     19      47   INGRESS_QOS_IPV6
<mark>0    6\7    320b   pmf-0       2035     13      40   INGRESS_ACL_L3_IPV6</mark>
</code>
</pre>
</div>

We could see the IPv4 ACL using 160 bits in the TCAM and IPv6 ACL using 320 bits.
At this moment, only default key is being used.

Let us add UDK

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit precedence port-range
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 09:12:51.495 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1902     97      30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1902     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1902     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1902     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1902     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
0    1      160b   pmf-0       1996     52      49   INGRESS_QOS_L2
0    2      160b   egress_acl  2031     17      17   EGRESS_QOS_MAP
0    3      160b   Free        2048     0       0    
0    4\5    320b   pmf-0       1999     27      31   INGRESS_LPTS_IPV6
<mark>0    4\5    320b   pmf-0       1999     3       39   INGRESS_ACL_L3_IPV4</mark>
</code>
</pre>
</div>

The above output shows, the configured UDK is taking precendence over default TCAM key. We can see the same ACL now uses 320 bits TCAM size. The key size programmed is as per the UDK defined to accomodate the various keys. The UDK has _src-addr dst-addr src-port dst-port proto packet-length frag-bit precedence port-range_. As mentioned above, each key has a size which get programmed in the TCAM.

```
src-addr  32 bits
dst-addr  32 bits
src-port  16 bits
dst-port  16 bits
proto      8 bits
packet-length  10 bits
frag-bit   3 bits
precedence 8 bits
port-range 24 bits

Total = 149 + ACL_ID (8 bits) and copy engines in TCAM
```

So the TCAM size needed to accomodate the UDK with these many keys needs to be more than 160 bits. 


Similarly let us check IPv6 ACL

```
hw-module profile tcam format access-list ipv6 src-addr src-port dst-addr dst-port next-hdr payload-length
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24(config)#interface tenGigE 0/0/0/0.10
RP/0/RP0/CPU0:N55-24(config-subif)#ipv6 access-group IPv6_ingress ingress 
RP/0/RP0/CPU0:N55-24(config-subif)#commit 
Sun Aug 16 09:16:04.486 UTC
<mark>% Failed to commit one or more configuration items during a pseudo-atomic operation. All changes made have been reverted. Please issue 'show configuration failed [inheritance]' from this session to view the errors</mark>
RP/0/RP0/CPU0:N55-24(config-subif)#show configuration failed 
Sun Aug 16 09:16:09.777 UTC
!! SEMANTIC ERRORS: This configuration was rejected by 
!! the system due to semantic errors. The individual 
!! errors with each failed configuration command can be 
!! found below.
interface TenGigE0/0/0/0.10
 ipv6 access-group IPv6_ingress ingress
<mark>!!% 'DPA' detected the 'warning' condition 'SDK - Table full'</mark>
!
end
</code>
</pre>
</div>

The above output shows the key size is not able to accomodate in the TCAM and hence the ACL is rejected. The same ACL was getting applied if we didnt use a UDK. Let us see the reason of getting rejected. 

```
src-addr  128 bits
dst-addr  128 bits
src-port  16 bits
dst-port  16 bits
next-header 8 bits
payload-length  16 bits

Total = 312 + ACL_ID (8 bits) and copy engines in TCAM
```
As we can see there is no space left for copy engines and TCAM space is almost full with the defined keys itself. So the users need to define the UDK carefully, one for the ACL to be configurable and second to utilize the TCAM resources wisely.

### How many UDK you can configure ?

You can configure only one UDK per location. If you try to configure another UDK, when one already exist for that location it will be overridden.

For example, we have this existing UDK

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit precedence port-range
hw-module profile tcam format access-list ipv6 src-addr src-port dst-addr dst-port next-hdr payload-length
```

Configuring another UDK for the same location

```
RP/0/RP0/CPU0:N55-24(config)#hw-module profile tcam format access-list ipv4 src-addr dst-addr packet-length 
Sun Aug 16 09:29:53.771 UTC
In order to activate/deactivate this ipv4 profile, you must manually reload the chassis/all line cards
RP/0/RP0/CPU0:N55-24(config)#hw-module profile tcam format access-list ipv6 src-addr dst-addr 
Sun Aug 16 09:30:16.648 UTC
In order to activate/deactivate this ipv6 profile, you must manually reload the chassis/all line cards
RP/0/RP0/CPU0:N55-24(config)#commit 
Sun Aug 16 09:30:22.250 UTC
RP/0/RP0/CPU0:N55-24(config)#
```

It overrides the previous UDK after reload

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr packet-length
hw-module profile tcam format access-list ipv6 src-addr src-port dst-addr next-hdr
```

### Configuring two different ACL's on different interface using same global UDK

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location all   
Sun Aug 16 13:42:36.260 UTC
<mark>Interface : TenGigE0/0/0/0.10 
    Input  ACL : Common-ACL : N/A  ACL : test-ipv4</mark>  
    Output ACL : N/A
<mark>Interface : TenGigE0/0/0/0.20 
    Input  ACL : Common-ACL : N/A  ACL : test-acl-v4-pkt-length</mark>  
    Output ACL : N/A
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show controllers npu internaltcam location 0/0/CPU0 
Sun Aug 16 13:08:36.481 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   pmf-0       1897     102     30   INGRESS_LPTS_IPV4
0    0      160b   pmf-0       1897     10      36   INGRESS_RX_ISIS
0    0      160b   pmf-0       1897     23      46   INGRESS_QOS_IPV4
0    0      160b   pmf-0       1897     15      48   INGRESS_QOS_MPLS
0    0      160b   pmf-0       1897     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    1      160b   pmf-0       1991     5       39   INGRESS_ACL_L3_IPV4</mark>
</code>
</pre>
</div>

From the above output, we can see there are 2 different ACL's applied and TCAM size occupied @160 bits


## Global or LC Specific UDK : Preference ?

In the above sections, we saw how a global UDK when defined, takes precedence over default key.
What happens when we define a Line Card specific UDK along with Global UDK ?

Let us see with the help of an example. We have a modular chassis with Line Card present in slot 1

```
RP/0/RP0/CPU0:N55-38#sho platform 
Mon Aug 17 13:11:50.976 UTC
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/1             NC55-MPA-2TH-S             DISABLED          
0/0/CPU0          NC55-MOD-A-S               IOS XR RUN        NSHUT
0/0/NPU0          Slice                      UP                
0/1/CPU0          NC55-18H18F                IOS XR RUN        NSHUT
0/1/NPU0          Slice                      UP                
0/1/NPU1          Slice                      UP                
0/1/NPU2          Slice                      UP                
0/RP0/CPU0        NC55-RP-E(Active)          IOS XR RUN        NSHUT
0/FC0             NC55-5504-FC               OPERATIONAL       NSHUT
0/FC1             NC55-5504-FC               OPERATIONAL       NSHUT
0/FC2             NC55-5504-FC               OPERATIONAL       NSHUT
0/FC3             NC55-5504-FC               OPERATIONAL       NSHUT
0/FC4             NC55-5504-FC               OPERATIONAL       NSHUT
0/FC5             NC55-5504-FC               OPERATIONAL       NSHUT
0/FT0             NC55-5504-FAN              OPERATIONAL       NSHUT
0/FT1             NC55-5504-FAN              OPERATIONAL       NSHUT
0/FT2             NC55-5504-FAN              OPERATIONAL       NSHUT
0/PM0             NC55-PWR-3KW-AC            OPERATIONAL       NSHUT
0/PM2             NC55-PWR-3KW-AC            OPERATIONAL       NSHUT
0/SC0             NC55-SC                    OPERATIONAL       NSHUT
0/SC1             NC55-SC                    OPERATIONAL       NSHUT
RP/0/RP0/CPU0:N55-38#

```

**UDK configured for 2 different location. If we dont specify the location it is considered 0/0/CPU0**


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port frag-bit location 0/0/CPU0
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port packet-length frag-bit location 0/1/CPU0</mark>
</code>
</pre>
</div>

The UDK for location 0/0/CPU doesnt include the key _packet-length_. The UDK for location 0/1/CPU0 includes that key. Let us apply the below policy on interfaces corresponding to those locations

```
ipv4 access-list test-acl-v4-pkt-length
 10 permit ipv4 any any packet-length range 800 831
```

**Applying the ACL on a interface at location 0/0/CPU0, we get the below error** 


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-38(config)#interface tenGigE 0/0/0/1    
RP/0/RP0/CPU0:N55-38(config-if)#ipv4 access-group test-acl-v4-pkt-length ingress
RP/0/RP0/CPU0:N55-38(config-if)#commit 
Mon Aug 17 13:22:35.250 UTC
<mark>LC/0/0/CPU0:Aug 17 13:22:35.332 UTC: pfilter_ea[146]: %PKT_INFRA-DPA_FM-3-USER_DEF_TCAM_KEY_PARAM_MISSING : ACL test-acl-v4-pkt-length, dir 0, seq 10, IPv4, 'dpa_feat_mgr' detected the 'warning' condition 'Parameter not programmed on ACL TCAM UDK (User Defined Key), check syslog for more details': Packet Length</mark> 
% Failed to commit one or more configuration items during a pseudo-atomic operation. All changes made have been reverted. Please issue 'show configuration failed [inheritance]' from this session to view the errors
RP/0/RP0/CPU0:N55-38(config-if)#show configuration failed 
Mon Aug 17 13:22:56.445 UTC
!! SEMANTIC ERRORS: This configuration was rejected by 
!! the system due to semantic errors. The individual 
!! errors with each failed configuration command can be 
!! found below.
<mark>interface TenGigE0/0/0/1
 ipv4 access-group test-acl-v4-pkt-length ingress
!!% 'dpa_feat_mgr' detected the 'warning' condition 'Parameter not programmed on ACL TCAM UDK (User Defined Key), check syslog for more details'</mark>
!
end
</code>
</pre>
</div>

**Applying the ACL on interface at location 0/1/CPU0.**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-38#show running-config interface hundredGigE 0/1/0/6
Mon Aug 17 13:29:46.432 UTC
interface HundredGigE0/1/0/6
 ipv4 address 106.1.1.1 255.255.255.0
 <mark>ipv4 access-group test-acl-v4-pkt-length ingress</mark>
!
</code>
</pre>
</div>

```
RP/0/RP0/CPU0:N55-38# show access-lists ipv4 test-acl-v4-pkt-length hardware ingress verify location 0/0/CPO</mark>
Mon Aug 17 14:29:49.576 UTC
Invalid ACL name or not attached in specified direction/interface

RP/0/RP0/CPU0:N55-38# show access-lists ipv4 test-acl-v4-pkt-length hardware i$
Mon Aug 17 14:29:59.391 UTC

Verifying TCAM entries for test-acl-v4-pkt-length
Please wait...                                   
                                                 
 
 
    INTF    NPU lookup  ACL # intf Total  compression Total   result failed(Entry) TCAM entries 
                type    ID  shared ACES   prefix-type Entries        ACE SEQ #     verified
 ---------- --- ------- --- ------ ------ ----------- ------- ------ ------------- ------------
 
HundredGigE0_1_0_6 (ifhandle: 0x8000a8)

              0 IPV4      2      1      1 NONE              2 passed                          2

```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-38#show controllers npu internaltcam location 0/1/CPU0
Mon Aug 17 14:28:03.398 UTC
Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   flp-tcam    2045     0       0    
0    1      160b   pmf-0       1993     38      30   INGRESS_LPTS_IPV4
0    1      160b   pmf-0       1993     12      36   INGRESS_RX_ISIS
0    1      160b   pmf-0       1993     2       46   INGRESS_QOS_IPV4
0    1      160b   pmf-0       1993     2       48   INGRESS_QOS_MPLS
0    1      160b   pmf-0       1993     1       54   INGRESS_EVPN_AA_ESI_TO_FBN_DB
<mark>0    2      160b   pmf-0       2036     3       39   INGRESS_ACL_L3_IPV4</mark>
</code>
</pre>
</div>

**Summary of TCAM key Precedence**

| Precedence Order                     |
|--------------------------------------|
| Line Card-specific UDK (if defined)  |
| Global UDK (if defined)              |
| Default TCAM Key                     |


### Reference

[CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/71x/b-ip-addresses-cg-ncs5500-71x/b-ip-addresses-cg-ncs5500-71x_chapter_0111.html "CCO Config Guide")

### Summary

In this document, we covered the details of User Defined Key - UDK for NCS55xx and NCS5xx. We also saw the advantages of using the UDK, particularly  optimize the valuable TCAM resources. How the UDK will take precedence when configured, over the default key. One thing to note is UDK can be used with the keys which are already defined. What if user wants to define their own fields and match against that.

Stay tuned for the next document on NCS55xx and NCS5xx UDF which will cover, how we can define fields and match the traffic against the same and apply action on it.
