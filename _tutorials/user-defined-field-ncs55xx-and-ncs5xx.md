---
published: true
date: '2020-09-02 15:39 +0530'
title: User Defined Field NCS55xx and NCS5xx
author: Tejas Lad
position: top
excerpt: This document aims to deep dive into UDF ACL for NCS55xx and NCS5xx ACL's
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

In this section, we will try to cover a few scenarios to show how we can use custom fields to match/filter traffic. We have taken simple examples to make the concept easier to understand. Users can take tips from this and can use their own way of classifying traffic based on their criterias.

### Matching layer 2 entities on a Layer 3 interface

- Consider the below toplogy

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%201.33.33%20PM.png)

We have a layer 3 connectivity end to end and host 60.1.1.2 wants to reach 70.1.1.2. In a normal scenario, we wont we able to match the VLAN ID on a layer 3 interface. For doing that we would need to define a Layer 2 ACL. But that wont be allowed to configure on a Layer 3 interface. Now what if a network administrator wants to match or filter a packet on his desired field (in this case consider VLAN id). If the platform does not have the capability to filter on deep packet analysis, network administrator will not be able to achieve that. Thanks to UDF capability of NCS55xx and NCS5xx, we can filter packets on user defined fields.

**Configuring UDF**

Consider the packet below. We are sending the packet with source address 60.1.1.2 and destination address 70.1.1.2 with an encapsulation of 10.

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%201.42.58%20PM.png)

If we want to filter the packet on the basis of encapsulation 10 we need to define the below UDF globally.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>udf udf_vlan header outer l2 offset 14 length 2
hw-module profile tcam format access-list ipv4 src-addr dst-addr udf1 udf_vlan</mark>
</code>
</pre>
</div>

Note: Reload of the router or line card will be needed after configuring or modifying the hw-module profile.
{: .notice--info}

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


### Matching Fragments using UDF


In previous technote for [Fragment Matching](https://xrdocs.io/ncs5500/tutorials/acl-ip-fragments-matching-ncs55xx-and-ncs5xx/ "Fragment Matching"), we saw there are 2 keywords: Fragments and Fragment-type. We saw that if we want to match initial fragments we need to use keyword Fragment-type, but this was supported on systems with external TCAM only. What if we have a system without external TCAM ? Thanks to UDF, we dont need to upgrade the whole system. Let us see, how we can achieve that. Consider the same topology as above.

We send the below fragmented packet with FO=0.

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%203.14.08%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%203.14.41%20PM.png)

As we have not applied any ACL, end to end traffic is flowing fine.

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%203.16.40%20PM.png)

For matching the Fragment Offset = 0, we will configure the below UDF

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>udf udf_frag header outer l3 offset 24 length 2
hw-module profile tcam format access-list ipv4 src-addr dst-addr udf1 udf_frag</mark>
</code>
</pre>
</div> 

**Referring the UDF to the ACL and applying it on the interface**

```
ipv4 access-list UDF_FRAG
 10 deny ipv4 any any udf udf_frag 0x2000 0xffff
 20 permit ipv4 any any

interface TenGigE0/0/0/0.10
 description using it for ACL testing
 ipv4 address 60.1.1.1 255.255.255.0
 ipv6 address 60::1/64
 load-interval 30
 encapsulation dot1q 10
 ipv4 access-group UDF_FRAG ingress

```

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%2010.45.44%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-08-31%20at%2010.53.21%20PM.png)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_FRAG hardware ingress location 0/0/CPU0        
Mon Aug 31 17:21:22.017 UTC
ipv4 access-list UDF_FRAG
<mark>10 deny ipv4 any any (5491101 matches)</mark>
 20 permit ipv4 any any
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>


### UDF Filtering for Layer 4 Header

The TCP header contains several one-bit boolean fields known as flags used to influence the flow of data across a TCP connection. In our example, we will try to set the URG flag. It is used to inform a receiving station that certain data within a segment is urgent and should be prioritized. If the URG flag is set, the receiving station evaluates the urgent pointer, a 16-bit field in the TCP header. This pointer indicates how much of the data in the segment, counting from the first byte, is urgent. [Reference](https://packetlife.net/blog/2011/mar/2/tcp-flags-psh-and-urg/#:~:text=The%20URG%20flag%20is%20used,field%20in%20the%20TCP%20header. "Reference")

![]({{site.baseurl}}/images/Screenshot%202020-09-02%20at%203.16.02%20PM.png)

**Configuring the UDF**

```
udf udf_l4 header outer l4 offset 13 length 1
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto frag-bit udf1 udf_l4

ipv4 access-list UDF_L4
 10 permit ipv4 any any udf udf_l4 0x21 0xff
```

In this example, we want to allow traffic with TCP urgent flag set and rest all traffic should be denied. As per the packet capture, we can see URG flag is set. Traffic is flowing end to end and the packets are matching ACE 10.

![]({{site.baseurl}}/images/Screenshot%202020-09-02%20at%203.17.26%20PM.png)

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_L4 hardware ingress location 0/0/CPU0 
Tue Sep  1 06:51:58.088 UTC
ipv4 access-list UDF_L4
 10 permit ipv4 any any (585576 matches)
RP/0/RP0/CPU0:N55-24#

```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UDF_L4 Details:
<mark>Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
ACL ID: 2
<mark>ACE Action: PERMIT</mark>
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 1101498
UDF Entries:
        1: udf_l4:   0x21       (mask: 0xff)</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8E97C0A8
Sequence Number: IMPLICIT DENY
NPU ID: 0
Number of DPA Entries: 1
ACL ID: 2
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
Hit Packet Count: 0
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8E97C888
</code>
</pre>
</div>

Let us change the traffic pattern with URG Flag unset.

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%2012.37.50%20PM.png)

```
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_L4 hardware ingress location 0/0/CPU0 
Tue Sep  1 07:04:41.524 UTC
ipv4 access-list UDF_L4
 10 permit ipv4 any any
RP/0/RP0/CPU0:N55-24#

```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
UDF_L4 Details:
<mark>Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 2</mark>
<mark>ACE Action: PERMIT</mark>
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 0
UDF Entries:
        1: udf_l4:   0x21       (mask: 0xff)</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8E97C0A8
<mark>Sequence Number: IMPLICIT DENY</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 2</mark>
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 545482</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8E97C888
</code>
</pre>
</div>

### UDF based ACL for traffic mirroring

One of the important use case for UDF based ACL is along with ERSPAN. ERSPAN feature allows you to monitor traffic on one or more ports. It is primarily for troubleshooting. The router encapsulates the traffic with ERSPAN header inside a GRE packet. It then sends this replicated monitor traffic to a destination IP address via a GRE tunnel. We will not go into more details on ERSPAN. For more details please [refer](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/lanswitch/configuration/xe-3s/lanswitch-xe-3s-book/lnsw-conf-erspan.html "refer").

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%205.37.48%20PM.png)

We will focus on how UDF ACL can be used along with ERSPAN to control the traffic which needs to be mirrored. UDF with ACL allows deeper packet analysis. ERSPAN ACL feature is used to mirror only interesting traffic. The main benefit is that it reduces the amount of traffic being mirrored and hence conserve bandwidth. Only ingress ACL is used for ERSPAN. Only Access Control Entries (ACEs) with capture keyword will be considered for mirroring. Both permit and deny packets will be captured if the ACE contains capture keyword. This feature is supported on all the platforms i.e NCS5xx and NCS55xx with J/J+/J2 asics.

User-Defined Field allows a customer to define a custom key by specifying the location and size of the field to match on. UDF ACL can be used to find patterns inside the inner header when packets are tunneled like is the case of ERSPAN where mirrored packets are going over a GRE Tunnel, or in the payload if the pattern is within the first 128 bytes of the packet.

**GRE Tunnel Configs**

```
interface tunnel-ip1
 ipv4 address 15.1.1.1 255.255.255.0
 tunnel mode gre ipv4
 tunnel source 172.16.3.24
 tunnel destination 70.1.1.2

```

**ERSPAN Configs**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>monitor-session 1 ethernet
 destination interface tunnel-ip1</mark>
</code>
</pre>
</div> 
 
<div class="highlighter-rouge">
<pre class="highlight">
<code> 
interface TenGigE0/0/0/0
 description *** To IXIA 1/3 *** 
 ipv4 address 100.24.0.1 255.255.255.252
 <mark>monitor-session 1 ethernet direction rx-only port-level</mark>
 </code>
</pre>
</div>

This will remotely span the traffic over a GRE tunnel 

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.36.30%20PM.png)

We can see we are sending 20000 packets (vlan 10 and 20) and receiving 40000 packets. This is a combination of data traffic plus spanned traffic 

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.37.35%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.37.53%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.37.44%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.38.01%20PM.png)


Now lets say for troubleshooting purpose, we only want traffic based on VLAN 10 to be mirrored. Other traffic should not be mirrored. It should be permitted and not denied. This can be achieved by configuring ACL along with ERSPAN. We need to apply the following configurations to achieve the same.

```
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto frag-bit udf1 udf_erspan
udf udf_erspan header outer l2 offset 14 length 2
```

**Referencing it to ACL and applying on the interface**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#
RP/0/RP0/CPU0:N55-24#show running-config interface tenGigE 0/0/0/0
Tue Sep  1 10:24:50.937 UTC
interface TenGigE0/0/0/0
 description *** To IXIA 1/3 *** 
 ipv4 address 100.24.0.1 255.255.255.252
 <mark>monitor-session 1 ethernet direction rx-only port-level acl</mark>
</code>
</pre>
</div>



<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>ipv4 access-list UDF_ERSPAN
 10 permit ipv4 any any udf udf_erspan 0xa 0xffff capture
 20 permit ipv4 any any</mark>
</code>
</pre>
</div>

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%203.52.38%20PM.png)

![]({{site.baseurl}}/images/Screenshot%202020-09-01%20at%204.00.28%20PM.png)

From the above output, we can see that the traffic is mirrored only for VLAN 10. The traffic for VLAN 20 is permitted but not mirrored. 


### UDF Matching on Inner Packet Header.

Let us see an example of how users can match the traffic pattern based on inner packet headers. 
We have generated a packet with custom payload.

![]({{site.baseurl}}/images/Screenshot%202020-09-02%20at%201.46.38%20PM.png)

User wants to match on the basis of information type in the custom payload. In our example, we have the information type as Information Request in the custome ICMP packet. For matching the traffic on basis of inner header, we can use the below UDF.

```
udf udf_custom header inner l3 offset 0 length 2
hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto frag-bit udf1 udf_custom
```

**Referencing the UDF to ACL and attaching it to the interface**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list UDF_CUSTOM
<mark> 10 permit ipv4 any any udf udf_custom 0xf00 0xffff
 20 deny ipv4 any any</mark>
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
 <mark>ipv4 access-group UDF_CUSTOM ingress</mark>
</code>
</pre>
</div>

We can see the traffic is flowing end to end and packets are matching the ACE.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_CUSTOM hardware ingress location 0/0/CPU0 
Wed Sep  2 08:24:51.944 UTC
ipv4 access-list UDF_CUSTOM
<mark> 10 permit ipv4 any any (301342 matches)</mark>
 20 deny ipv4 any any
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_CUSTOM hardware ingress detail location 0/0/CPU0 
Wed Sep  2 09:05:30.678 UTC
UDF_CUSTOM Details:
<mark>Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1</mark>
ACE Action: PERMIT
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 1188455
UDF Entries:
        1: udf_custom:   0xf00  (mask: 0xffff)</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8EC490A8
<mark>Sequence Number: 20</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1</mark>
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 0</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8EC49498
<mark>Sequence Number: IMPLICIT DENY</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1</mark>
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 0</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8EC49888
</code>
</pre>
</div> 

![]({{site.baseurl}}/images/Screenshot%202020-09-02%20at%201.57.43%20PM.png)

If we change the traffic pattern and remove the custom payload the traffic doesnt match the sequence 10 and traffic is denied.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 UDF_CUSTOM hardware ingress location 0/0/CPU0 
Wed Sep  2 09:11:51.588 UTC
ipv4 access-list UDF_CUSTOM
 10 permit ipv4 any any
 <mark>20 deny ipv4 any any (44974 matches)</mark>
</code>
</pre>
</div>

### UDF ACL for matching DVMRP packets

The Distance Vector Multicast Routing Protocol or DVMRP, is a routing protocol used to share information between routers to facilitate the transportation of IP multicast packets among networks [refer](https://en.wikipedia.org/wiki/Distance_Vector_Multicast_Routing_Protocol "refer"). DVMRP uses the Internet Group Management Protocol (IGMP) to exchange
routing datagrams. To know further details on DVMRP please refer [RFC 1075](https://tools.ietf.org/html/rfc1075 "RFC 1075"). DVMRP DDoS can cause IGMP process crash and also potentially bring down the router. Thanks to NCS55xx and NCS5xx capability to filter the DVMRP packets with UDF, we can protect the network with such attacks. UDF can be used to match the DVMRP packets and drop it at the ingress interface.

**Packet with DVMRP header**

![Screenshot 2020-09-07 at 5.30.40 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-07 at 5.30.40 PM.png)

**Configuring UDF and referencing it to the ACL**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>udf udf_dvmrp header outer l4 offset 0 length 1
hw-module profile tcam format access-list ipv4 src-addr dst-addr proto frag-bit udf1 udf_dvmrp</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list UDF_DVMRP
<mark>10 deny igmp any any udf udf_dvmrp 0x13 0xff</mark>
20 permit ipv4 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
interface Bundle-Ether31000
ipv4 address 1.1.1.2 255.255.255.0
<mark>ipv4 access-group UDF_DVMRP ingress</mark>
</code>
</pre>
</div>


We can see that ACL is matching the ingress DVMRP packets

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:xrg-307-NCS-5501-SE#show access-lists ipv4 UDF_DVMRP hardware ingress location 0/0/CPU0
Thu Sep 3 23:20:03.056 UTC
ipv4 access-list UDF_DVMRP
<mark>10 deny igmp any any (1799642834 matches)</mark> 
20 permit ipv4 any any
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:xrg-307-NCS-5501-SE#show access-lists ipv4 UDF_DVMRP hardware ingress det location 0/0/CPU0
UDF_DVMRP Details:
<mark>Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1</mark>
<mark>ACE Action: DENY</mark>
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
<mark>Hit Packet Count: 1819195451</mark>      
UDF Entries:
1: udf_dvmrp: 0x13 (mask: 0xff)
DPA Entry: 1
Entry Index: 0
DPA Handle: 0x8CD870A8
<mark>Sequence Number: 20</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1</mark>
<mark>ACE Action: PERMIT</mark>
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
DPA Entry: 1
Entry Index: 0
DPA Handle: 0x8C4FBAC8
Sequence Number: IMPLICIT DENY
NPU ID: 0
Number of DPA Entries: 1
ACL ID: 1
ACE Action: DENY
ACE Logging: DISABLED
ABF Action: 0(ABF_NONE)
Hit Packet Count: 0
DPA Entry: 1
Entry Index: 0
DPA Handle: 0x8C4FC4E8
</code>
</pre>
</div>

Thanks to Santosh Sharma for helping out test this scenario.
{: .notice--success}


## References

  - [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/72x/b-ip-addresses-cg-ncs5500-72x/m-implementing-access-lists-prefix-lists-ncs5500.html "CCO Config Guide")


## Summary

In this tech-note, we saw how we can overcome the limits of static matching criterias by using user defined fields. This gives the users capabilities of matching the packets dynamically as per the requirement. This is very useful in troubleshooting network issues and identifying packets. We saw a few simple examples with UDF. These can be customized as per the network requirements. With the availability of UDF, it is possible for the NCS55xx and NCS5xx platform to be more capable and flexible when fulfilling the packet classification functionalities.
