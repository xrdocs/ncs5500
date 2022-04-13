---
published: true
date: '2020-09-08 14:36 +0530'
title: ACL Based Forwarding and Object Tracking for NCS5xx and NCS55xx
author: Tejas Lad
excerpt: >-
  This document aims to cover the concept of ABF and the used of object tracking
  with ABF
tags:
  - NCS5500
  - NCS500
  - NCS5xx
  - NCS55xx
  - ACL
  - ABF
  - ABF_OT
position: top
---
{% include toc %}

## Introduction

In today’s converged networks, operators have to implement packet forwarding in a way that goes beyond traditional routing protocol. Sometimes operators want certain traffic to be engineered to a separate path based on certain rules. They do not want it to take the path computated by the dynamic routing protocols. ACL Based Forwarding (ABF) can be used as a technique to achieve the same. ABF is a subset of PBR (Policy Based Routing) infrastructure in the XR platform. ABF allows traffic matching specific ACL rule to be forwarded to user specified nexthop instead of route selected by a routing protocol. ABF does not modify anything in the packet itself and therefore only affects the local routing/forwarding decision. The path for reaching the ABF next-hop however, is determined by the normal routing/forwarding table.

![Screenshot 2020-09-08 at 3.23.09 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 3.23.09 PM.png)

## Different Types of ABF 

There are 3 types of ABF supported on NCS55xx and NCS5xx [Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/72x/b-ip-addresses-cg-ncs5500-72x/m-implementing-access-lists-prefix-lists-ncs5500.html "Reference")

  - **ABF for IPv4/IPv6**
  
    - Only the next-hop IP address is specified in the ACE rule.
	- The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - Default VRF is used in this type of ABF.
    
    ![Screenshot 2020-09-08 at 4.21.12 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.12 PM.png)


  - **VRF-aware ABF for IPv4/IPv6**
  
    - Both the next-hop IP address and the next-hop VRF are specified in the ACE rule.
    - The matching traffic is forwarded to the first "up" next-hop, as specified in the ACE. 
    - The specified VRF is used instead of the default VRF for determining the path.
    
    ![Screenshot 2020-09-08 at 4.21.19 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.19 PM.png)


  - **VRF-select ABF for IPv4/IPv6** 
  
    - Only the next-hop VRF is specified in the ACE rule.
    - The matching traffic is forwarded to the first “UP” VRF as specified in the ACE rule.
    - Supported from IOS-XR 6.5.1 onwards.
    
    ![Screenshot 2020-09-08 at 4.21.26 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 4.21.26 PM.png)


## Possible Use cases of ABF 
   (Reference 1)

**Source-Based Transit Provider Selection** – Internet service providers and other organizations can use ABF to route traffic originating from different sets of users through different internet connections across the policy routers.

**Cost Savings** – Organizations can achieve cost savings by distributing interactive and batch traffic among low-bandwidth, low-cost permanent paths and high-bandwidth, high-cost, switched paths.

**Load Sharing** – In addition to the dynamic load-sharing capabilities offered by destination-based routing that the Cisco IOS-XR software provides network manager can implement policies to distribute traffic among multiple paths based on the traffic characteristics.


## ABF Feature Support

- ABF is supported only in Ingress Direction. 
- ABF is not supported in the Egress direction
- ABF is supported only for IPv4 and IPv6. 
- ABF is not supported for L2 ACL.
- ABF is supported upto only 3 next-hops. The next-hops are selected on the basis of its configuration in the ACE.
- It is supported only for permit action. 
- Deny action is not supported with ABF.
- ABF is supported on Physical Interfaces, Sub-interfaces, Bundle Interfaces and Bundle Sub-Interfaces.
- ABF is supported for common-ACL's from IOS-XR 7.6.1
- ABF supports dynamic next-hop modifications.
- ABF default route is not supported.
- IPv4 ABF next hops routed over GRE interfaces is supported.

## ABF Platform Implementation 

Now that we have understood the feature and its use cases, let us jump into the configuration and implementaion on NCS55xx and NCS5xx. We will be a taking a very simple topology for easier illustration of the feature.

![Screenshot 2020-09-08 at 7.11.34 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 7.11.34 PM.png)

In the above topology, we have network 70.1.1.0/24 behind router R5 and network 60.1.1.0/24 behind router R1. We have configured two ISIS neighbors, but the path calculated is just via TenGig 0/0/0/6 as we have higher metric on TenGig 0/0/0/7


<div class="highlighter-rouge">
<pre class="highlight">
<code>
<mark>RP/0/RP0/CPU0:N55-24#show isis neighbors 
IS-IS 1 neighbors:
System Id      Interface        SNPA           State Holdtime Type IETF-NSF
N540-49        Te0/0/0/7        *PtoP*         Up    26       L2   Capable 
N540-49        Te0/0/0/6        *PtoP*         Up    28       L2   Capable</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show route 70.1.1.2
Routing entry for 70.1.1.0/24
  Known via "isis 1", distance 115, metric 20, type level-2
  Installed Sep  8 08:19:09.054 for 05:30:18
  Routing Descriptor Blocks
   <mark>65.1.1.2, from 172.16.4.49, via TenGigE0/0/0/6</mark>
      Route metric is 20
  No advertising protos. 
</code>
</pre>
</div>

As we mentioned above, let us apply an ABF to influence the critical traffic to take a path as per our configured next-hop.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list ABF_Test
 <mark>10 permit ipv4 any any dscp af11 nexthop1 ipv4 65.1.1.2 nexthop2 ipv4 70.1.1.1
 20 permit ipv4 any any dscp ef nexthop1 ipv4 66.1.1.2 nexthop2 ipv4 70.1.1.1</mark>
</code>
</pre>
</div>

This ACL will sent the traffic destined for 70.1.1.2 with DSCF AF11 via interface TenGig 0/0/0/6 and traffic with DSCP EF (critical traffic) via TenGig 0/0/0/7. Let us verify the same.

## ABF Verification

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 usage pfilter location 0/0/CPU0 
<mark>Interface : TenGigE0/0/0/0.10 
    Input  ACL : Common-ACL : N/A  ACL : ABF_Test</mark>  
    Output ACL : N/A
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress location 0/0/CPU0 
<mark>ipv4 access-list ABF_Test
 10 permit ipv4 any any dscp af11 (next-hop: addr=65.1.1.2, vrf name=default)
 20 permit ipv4 any any dscp ef (next-hop: addr=66.1.1.2, vrf name=default)</mark>
</code>
</pre>
</div>

![Screenshot 2020-09-08 at 8.05.04 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 8.05.04 PM.png)


![Screenshot 2020-09-08 at 8.19.07 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 8.19.07 PM.png) 

We can see the traffic (10000 packets of AF11 and 20000 packets of EF) is flowing fine. 

![Screenshot 2020-09-08 at 11.05.55 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-08 at 11.05.55 PM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress location 0/0/CPU0 
ipv4 access-list ABF_Test
 <mark>10 permit ipv4 any any dscp af11 (99377687 matches) (next-hop: addr=65.1.1.2, vrf name=default)
 20 permit ipv4 any any dscp ef (198755422 matches) (next-hop: addr=66.1.1.2, vrf name=default)</mark>
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show access-lists ipv4 ABF_Test hardware ingress detail location 0/0/CPU0 
<mark>ABF_Test Details:
Sequence Number: 10</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1
ACE Action: PERMIT</mark>
ACE Logging: DISABLED
<mark>ABF Action: 1(ABF_NH)
ABF IP: 65.1.1.2
ABF VRF: default</mark>
ABF FEC ID: 0x2001ffb8
<mark>Hit Packet Count: 99688849</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8ED2D0A8
        DSCP: 0x28 (Mask 0xFC)
<mark>Sequence Number: 20</mark>
NPU ID: 0
Number of DPA Entries: 1
<mark>ACL ID: 1
ACE Action: PERMIT</mark>
ACE Logging: DISABLED
<mark>ABF Action: 1(ABF_NH)
ABF IP: 66.1.1.2
ABF VRF: default</mark>
ABF FEC ID: 0x2001ffb9
<mark>Hit Packet Count: 199377727</mark>
DPA Entry: 1
        Entry Index: 0
        DPA Handle: 0x8ED2D498
        DSCP: 0xB8 (Mask 0xFC)
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
        DPA Handle: 0x8ED2D888
</code>
</pre>
</div>

From the above outputs, we could successfully influence the traffic to take a path which is not installed in the routing table. This can be used by the operators for diverting the traffic to load balance or for purpose of troubleshooting.



## For Punt Packets 

- Packets punted in the ingress direction from the NPU to the linecard CPU are not subjected to ABF treatment due to lack of ABF support in the slow path. These packets will be forwarded normally based on destination-address lookup by the software dataplane. 
- Some examples of these types of packets are (but are not limited to) packets with IPv4 options, IPv6 extension headers, and packets destined for glean (unresolved/incomplete) adjacencies.

- Packets destined to the local IP interface ("for-us" packets) are subjected to redirect if they match the rule containing the ABF action. 
- This can be avoided by either designing the rule to be specific enough to avoid matching the “for-us” packets or placing an explicit permit ACE rule (with higher priority) into the ACL before the matching ABF rule.
[Reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/72x/b-ip-addresses-cg-ncs5500-72x/m-implementing-access-lists-prefix-lists-ncs5500.html "Reference")

## Resource Utilization 

As we already know the TCAM is very important resource in NCS55xx and NCS5xx. It needs to be optimized. The advantage of ABF, is that it is built within the same feature framework. ABF shares the same lookup field groups as ACL hereby saving the TCAM resources for additional lookups. Another advantage is ABF uses the CLI which is based on existing ACL infrastructure making it comfortable for the users to configure and implement. For further information on resource utilization please [refer](https://xrdocs.io/ncs5500/tutorials/security-acl-on-ncs5500-part1/ "refer")

## Important considerations for ABF Next Hop

While configuring the ABF next-hop we should consider the below:

  - The next-hop address cannot be of the loopback interface.
  - The next-hop address cannot an address routed/forwarded over the loopback interface.
  - The next-hop address caanot be the address of a local port.

## Object Tracking with ABF

In some scenarios, the ABF may fail to recognise that the next hop is not reachable and will keep on forwarding the packet to that next hop. This will cause traffic drop end to end. Consider the same topology as above. Traffic is flowing fine as per the configured next-hop. 

![Screenshot 2020-09-09 at 10.34.52 AM.png]({{site.baseurl}}/images/Screenshot 2020-09-09 at 10.34.52 AM.png)

Due to unknown reason, the link between the switch and R5 has gone down. Router R1 has no visibility of this and it will continue forwarding the traffic out of the interface Ten 0/0/0/7 as that link is in UP state. How to deal with this type of failure scenario ?

This is where we need Object-Tracking along with ABF. Track option in ABF enables track object to 
be specified along with nexthop ip address. Let us see this with configuration example.


```
ipv4 access-list ABF_Test
 10 permit ipv4 any any dscp ef nexthop1 ipv4 66.1.1.2 nexthop2 ipv4 65.1.1.2
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show interfaces tenGigE 0/0/0/7 | in rate 
Wed Sep  9 05:12:58.069 UTC
  30 second input rate 1000 bits/sec, 0 packets/sec
  30 second output rate 79361000 bits/sec, <mark>10000 packets/sec</mark>
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

**Shutting the interface between R5 and Switch**

We can see the traffic has dropped completely 

![Screenshot 2020-09-09 at 10.50.05 AM.png]({{site.baseurl}}/images/Screenshot 2020-09-09 at 10.50.05 AM.png)

The traffic is being forwarded over TenGig 0/0/0/7, as R1 has no visibility of the link down.

```
RP/0/RP0/CPU0:N55-24#show interfaces tenGigE 0/0/0/7 | i rate 
Wed Sep  9 05:18:29.082 UTC
  30 second input rate 0 bits/sec, 0 packets/sec
  30 second output rate 81244000 bits/sec, 10238 packets/sec
RP/0/RP0/CPU0:N55-24#

```

**Configuring object tracking for ABF**

```
ipsla
 operation 1
  type icmp echo
   destination address 66.1.1.2
   timeout 5000
   frequency 5
  !
 !
 schedule operation 1
  start-time now
 !
!

```

**Modifying the ACL to track the next hop**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ipv4 access-list ABF_Test
 <mark>10 permit ipv4 any any dscp ef nexthop1 track 1 ipv4 66.1.1.2 nexthop2 ipv4 65.1.1.2</mark>
</code>
</pre>
</div>

![Screenshot 2020-09-09 at 11.10.02 AM.png]({{site.baseurl}}/images/Screenshot 2020-09-09 at 11.10.02 AM.png)


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show ipsla statistics 
Entry number: 1 
    Modification time: 05:22:50.959 UTC Wed Sep 09 2020
    Start time       : 14:07:07.439 UTC Tue Sep 08 2020
    Number of operations attempted: 708
    Number of operations skipped  : 12
    Current seconds left in Life  : 0
    Operational state of entry    : Inactive
    Operational frequency(seconds): 5
    <mark>Connection loss occurred      : FALSE
    Timeout occurred              : FALSE</mark>
    Latest RTT (milliseconds)     : 1
    Latest operation start time   : 15:07:02.440 UTC Tue Sep 08 2020
    Next operation start time     : Inactive
    Latest operation return code  : OK
    RTT Values:
      RTTAvg  : 1          RTTMin: 1          RTTMax : 1         
      NumOfRTT: 1          RTTSum: 1          RTTSum2: 1
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show track 1
Track 1 
        Response Time Reporter 1 reachability 
        <mark>Reachability is UP</mark>
        2 changes, last change 14:09:23 UTC Tue Sep 08 2020
        Latest operation return code: OK 
        Latest RTT (millisecs) : 1 
</code>
</pre>
</div>


**Shutting the interface between R5 and Switch again**

```
RP/0/RP0/CPU0:N55-24#RP/0/RP0/CPU0:Sep  9 05:45:12.000 UTC: object_tracking[360]: %SERVICES-OT-6-TRACK_INFO : track 1 state Track_Down
```

![Screenshot 2020-09-09 at 11.19.05 AM.png]({{site.baseurl}}/images/Screenshot 2020-09-09 at 11.19.05 AM.png)

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show ipsla statistics 
Entry number: 1 
    Modification time: 05:44:45.008 UTC Wed Sep 09 2020
    Start time       : 05:44:45.011 UTC Wed Sep 09 2020
    Number of operations attempted: 13
    Number of operations skipped  : 13
    Current seconds left in Life  : 3470
    Operational state of entry    : Active
    Operational frequency(seconds): 5
    Connection loss occurred      : FALSE
    <mark>Timeout occurred              : TRUE</mark>
    Latest RTT (milliseconds)     : Unknown
    Latest operation start time   : 05:46:45.012 UTC Wed Sep 09 2020
    Next operation start time     : 05:46:50.012 UTC Wed Sep 09 2020
    Latest operation return code  : TimeOut
    RTT Values:
      RTTAvg  : 0          RTTMin: 0          RTTMax : 0         
      NumOfRTT: 0          RTTSum: 0          RTTSum2: 0
</code>
</pre>
</div>


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show track 1
Track 1 
        Response Time Reporter 1 reachability 
        <mark>Reachability is DOWN</mark>
        3 changes, last change 05:45:12 UTC Wed Sep 09 2020
        Latest operation return code: TimeOut 
        Latest RTT (millisecs) : 0 
RP/0/RP0/CPU0:N55-24#
</code>
</pre>
</div>

**Traffic is moved to next UP next-hop**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show interfaces tenGigE 0/0/0/6 | i rate 
Wed Sep  9 05:48:03.242 UTC
  30 second input rate 1000 bits/sec, 0 packets/sec
  <mark>30 second output rate 79362000 bits/sec, 10000 packets/sec</mark>
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show interfaces tenGigE 0/0/0/7 | i rate 
Wed Sep  9 05:47:58.801 UTC
  30 second input rate 0 bits/sec, 0 packets/sec
  <mark>30 second output rate 1000 bits/sec, 0 packets/sec</mark>
</code>
</pre>
</div>


That's it! 
{: .notice--success}

This is just one of scenario. There can be multiple sceanrios where network admins can identify the need of using object tracking along with ABF to prevent the loss of traffic.

## Reference

  - Reference 1: https://community.cisco.com/t5/service-providers-documents/asr9000-xr-abf-acl-based-forwarding/ta-p/3153403
  - [CCO Config Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/76x/b-ip-addresses-cg-ncs5500-76x/m-implementing-access-lists-prefix-lists-ncs5500.html)
  
  
## Summary

In this technote, we tried to cover the ABF feature, possible use cases and how to use ABF along with Object tracking to track the next-hops for failure scenarios. We also saw configuration examples on NCS55xx and NCS5xx. We have taken a simple network topology to explain the concept in easier way, but there is no restriction for ABF to work flawlessly in complex networks. Network admins can use this feature effectively to engineer the traffic along desired paths and perform load-balancing and troubleshooting as and when required. However we should note that since ABF is ACL-based, all packets which do not match an existing rules in the ACL will be subject to the default ACL rule i.e. drop all. Therefore, it is suggested that the user put an explicit rule which is of lower priority to "permit" all traffic.  This will ensure that all traffic which does not match an ABF rule will be permitted and forwarded as normal.
