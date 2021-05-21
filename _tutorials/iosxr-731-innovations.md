---
published: true
date: '2021-03-29 12:45 +0200'
title: IOS XR 7.3.1 Innovations in NCS5500/NCS5700/NCS500 Platforms
author: Nicolas Fevrier
excerpt: What's new in IOS XR731 in term of features and newly supported hardware
position: top
tags:
  - iosxr
---
{% include toc icon="table" title="XR 7.3.1 Innovations" %} 

## Introduction

IOS XR 7.3.1 has been published in late February 2021 and is an ED version for many XR platforms including:  
- NCS5500
- NCS5700
- NCS540
- NCS560

Release notes:
- NCS540: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/73x/b-release-notes-ncs540-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/73x/b-release-notes-ncs540-r731.html)  
- NCS560: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/73x/b-release-notes-ncs560-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/73x/b-release-notes-ncs560-r731.html)
- NCS5500: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html)

Software download for NCS5500:  
[https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/73x/release/notes/b-release-notes-ncs5500-r731.html)

![download731.png]({{site.baseurl}}/images/download731.png){: .align-center}

We asked a few colleagues to help documenting the new improvements brought by the 7.3.1 version. It will be split in software innovations on one side (Segment Routing, SRv6, EVPN, Multicast, QoS, Security, ...) and new supported hardware (chassis, power supply, new line cards and new fixed-form-factor products).

## Segment Routing IPv6 (SRv6)

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/Dna59DZlLmQ?autoplay=1" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jakub Horn presents and demonstrates in his lab:
- SRv6 uSID
- uSID config
- uSID ISIS config
- uSID BGP L3VPN config
- TI-LFA and uLoop Avoidance config
- Performance Measurement config
- FlexAlgo config

The configuration used for the demos are available here:  
[https://www.segment-routing.net/tutorials/srv6-731-features/](https://www.segment-routing.net/tutorials/srv6-731-features/)

## EVPN

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/3eoDB7uia6k" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jiri Chaloupka and Lampros Gkavogiannis detail the following features:  

- EVPN Head-End

EVPN control plane used on top of the head-end to decide which interface is active or stand-by.

![EVPN01.png]({{site.baseurl}}/images/EVPN01.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>evpn
 interface PW-Ether 1
  ethernet-segment
   identifier type 0 9.8.7.6.5.4.3.2.1

l2vpn
 xconnect group xc100
  p2p evpn-headend
   interface PW-Ether1
   neighbor evpn evi 1 target 1 source 1</code>
</pre>
</div>

- EVPN Convergence
  - Fast ReRoute
  
  Handled by the transport layer (SR TI-LFA for example)
  
  - MAC Mobility
  
  Available since EVPN inception, it uses a sequence number in the advertisement, to track VM move for example:
  
  ![EPNV02.png]({{site.baseurl}}/images/EPNV02.png){: .align-center}

  - Mass Withdraw
  
  Also available since the early EVPN days. In this example, CE1 is multihomed so PE3 sees the mac addresses through the same segment ESI1.
   
  ![EVPN03.png]({{site.baseurl}}/images/EVPN03.png){: .align-center}

PE1 losing the link to CE1, it just sends a RT1 update to PE3, that can reprogram the decision to send the traffic to PE2. But it takes a bit of time to generate, receive and compute this BGP advertisement. Hence the introduction of the next feature.
  
  - Edge Failure Fast ReRoute
  
  Similar to the L3VPN BGP PIC Edge feature, adapted for EVPN.
  
  ![EVPN04-.png]({{site.baseurl}}/images/EVPN04-.png){: .align-center}
  
  Using EBGP betwen CE and PE. PE1 will pre-program the back up path over PE2 that will forward via PE1 until the network fully converged. During that short period of time, we will have a sub-optimal routing path, but no packet loss.
  
  And it works for both All-Active and Single-Active.
  
  ![EVPN05.png]({{site.baseurl}}/images/EVPN05.png){: .align-center}

All-Active configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>evpn
 interface Bundle-Ether100
  ethernet-segment
   identifier type 0 36.37.36.37.36.37.36.37.01
   convergence
    reroute</code>
</pre>
</div>

Single-Active configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>evpn
 interface Bundle-Ether100
  ethernet-segment
   identifier type 0 36.37.36.37.36.37.36.37.01
   load-balancing-mode single-active
   convergence
    reroute</code>
</pre>
</div>

- EVPN Load Balancing

![EVPN06.png]({{site.baseurl}}/images/EVPN06.png){: .align-center}

Since the beginning we have All-Active mode where PE1 and PE2 are configured to make CE1 believe it's connected to a single device via link aggregation.  
Later, we introduced the single-active mode, followed by the port-active mode.

- SFA Single Flow Active

To improve the convergence, particularly when connected to legacy L2 protocols (like spanning-tree, REP-AG.G.8032), we introduce this new load balancing mode.

![EVPN07.png]({{site.baseurl}}/images/EVPN07.png){: .align-center}

SFA is leveraging pre-programmed information between PE1 and PE2.

<div class="highlighter-rouge">
<pre class="highlight">
<code>evpn
 interface Bundle-Ether100
  ethernet-segment
   identifier type 0 36.37.36.37.36.37.36.37.01
   load-balancing-mode single-flow-active
   convergence
    mac-mobility</code>
</pre>
</div>

![EVPN08.png]({{site.baseurl}}/images/EVPN08.png){: .align-center}

Check [https://datatracker.ietf.org/doc/html/draft-brissette-bess-evpn-l2gw-proto-06](https://datatracker.ietf.org/doc/html/draft-brissette-bess-evpn-l2gw-proto-06) for more details.

- Next-Hop Tracking for DF Election

We are enabling next-hop tracking for Route-Type 4. When NH disappears from the table, we trigger immediately a DF election.

<div class="highlighter-rouge">
<pre class="highlight">
<code>evpn
 interface Bundle-Ether100
  ethernet-segment
   identifier type 0 36.37.36.37.36.37.36.37.01
   load-balancing-mode single-active
   convergence
    <mark>nexthop-tracking</mark>
    reroute</code>
</pre>
</div>

![EVPN09.png]({{site.baseurl}}/images/EVPN09.png){: .align-center}

- NTP sync

When you have two nodes on the same ESI segment, and these two routers are NTP synchronized (no need for additional configuration). It will add the timestamps in the RT-4 and the whole convergence process will speed up.

- Multicast MultiHoming

![EVPN-mcast.png]({{site.baseurl}}/images/EVPN-mcast.png){: .align-center}

This new feature enables redundancy for multi-attached multicast receivers. It leverages Route Type 7 and 8 for IGMP Synchronization between PEs. Also, it requires Designated Forwarder (DF) Election between these PE routers.  
After IGMP snooping has been enabled and this information has been synced with the peer, both the peers need to act like a last hop router and send PIM join upstream. Once traffic arrives on both the peers, only one should forward it to the receiver. Designated Forwarder Election elects one peer to do the forwarding.

EVPN IPv4 MC Enhanced DF election for mcast: As per RFC 7432, designated forwarder (DF) election is at the granularity of <ESI,EVI> . However, customers need multi tenancy at finer level. 
A per multicast stream DF election is required, as they use a single Vlan for IPTV service

## Quality of Service

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/lvhHNdXEC_c" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Paban Sarma provides details on new QoS features:  
- Shared-Policy Instances

The purpose is to get aggregated QoS value for customers owning multiple sub-interfaces over one given physical port. Example: Customer A paid for 5Gbps on 3 sub-interfaces and Customer B, they paid for 3Gbps over 2 sub-interfaces.

![QoS1.png]({{site.baseurl}}/images/QoS1.png){: .align-center}

Limited to the same parent interface (physical or bundle) and is available for both ingress policers and egress queueing/shaper.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map spi-in
 class class-default
  police rate 500 mbps
  !
 !
 end-policy-map
!
policy-map spi-out
 class class-default
  shape average 500 mbps
 !
 end-policy-map
!
interface TenGigE0/0/0/16.1001 l2transport
 encapsulation dot1q 1001
 service-policy input spi-in shared-policy-instance spi-1-in
 service-policy output spi-out shared-policy-instance spi-1-out
!
interface TenGigE0/0/0/16.1002 l2transport
 encapsulation dot1q 1002
 service-policy input spi-in shared-policy-instance spi-1-in
 service-policy output spi-out shared-policy-instance spi-1-out
!</code>
</pre>
</div>

- Policy-map templates and uniqueness

This new feature is an enhancement of the unique policy-map scale.

**Before IOS XR 7.3.1**: each policy-map has an ID (total unique ID is 250): same policy map may be attached to different interface but only 250 unique ingress policy can exist on the system.  

**Starting with IOS XR 7.3.1** several policy-maps can share the same ID if they have:  
  Same classification (class-map name independent)  
  Same action policing (police rate independent)  
  Same marking action  

Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map P1-100M
class class-default
  police rate 100 mbps
  !
  set traffic-class 5
!
end-policy-map
!
policy-map P2-1000M
class class-default
  police rate 1000 mbps
  !
  set traffic-class 5
!
end-policy-map</code>
</pre>
</div>

P1-100M and P2-1000M have the same classification, same action policing (even if it's a policing at different rates) and same marking action: they are using the same ID and count for 1.

Before 7.3.1, we verify it uses two entries:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Router-721#show feature-mgr client qos-ea feature-info summary loc 0/0/CPU0 
NPU DIR      Lookup-type     ACL-ID Refcnt Feature-Name
--- --- -------------------- ------ ------ ------------
0   IN  L2_QOS               17     1      <mark>P1-100M:0</mark>
0   IN  L2_QOS               18     1      <mark>P2-1000M:0</mark>
RP/0/RP0/CPU0:Router-721#</code>
</pre>
</div>

With 7.3.1, it's only a single "feature-name"
  
<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Router-731#show feature-mgr client qos-ea feature-info summary  loc 0/0/CPU0       
NPU DIR      Lookup-type     ACL-ID Refcnt Feature-Name                     PolicyMap-Name
--- --- -------------------- ------ ------ -------------------------------  --------------
0   IN  L2_QOS               10     2      32b51d8e63702738b16423f7e8df7be7 <mark>P1-100M</mark>
                                                                            <mark>P2-1000M</mark>
RP/0/RP0/CPU0:Router-731#</code>
</pre>
</div>

For reference, let's list a couple of examples that don't share a common policy ID and count as 3 different entries:
  
<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map P1-100M
class class-default
  police rate 100 mbps
  !
  set traffic-class 5
!
end-policy-map
!
policy-map P2-1000M
class class-default
  police rate 1000 mbps
  !
  set traffic-class 3
!
end-policy-map
!
policy-map P3-1000M
class class-default
  police rate 1000 mbps
  !
  set traffic-class 3
  set cos 3
!
end-policy-map</code>
</pre>
</div>

Here, P1 and P2 are different, because setting different traffic-class values (3 and 5).  
Also, P3 is different than P1 and P2, because it sets cos 3 (and not the others).  
So, these 3 policy-maps are using 3 different IDs and are considered unique.

## Security

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/O6tTu2Nidj8" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Rakesh Kandula presents the latest improvements in IOS XR security brought in IOS XR 7.3.1:  
- SSD encryption (note: it's limited to the platforms running XR7)

Feature developed for use-cases like: router theft, RMA scenario, unauthorized disk swap and router decommissionning.  
Provides data-at-rest protection for sensitive data like running configuration
Encryption key is generated in hardware TAm (Trust Anchor module) chip and is unique per line card in a chassis, it encrypts only a disk partition and not the entire SSD.

Configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>disk encryption activate location 0/x/CPU0</code>
</pre>
</div>

- X.509v3 Certificate based SSH authentication

This new feature will remove the need for Username/Password management (no need for RSA key management for end users): authenticate end users logging into routers using x.509v3 certificates (based on RFC6187).  
Once local authentication passes, authorization can be local or remote (TACACS)

![X509v3-1.png]({{site.baseurl}}/images/X509v3-1.png){: .align-center}

1- End user presents their certificate to the router for authentication  
2- Router validates the certificate locally for authentication  
3- Authorization request is then sent to TACACS server to fetch the group info of the user  
4- User gets access to the router with the appropriate privilege level  

Configuration

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-1-731(config)#ssh server algorithms host-key x509v3-ssh-rsa
RP/0/RP0/CPU0:5508-1-731(config)#ssh server trustpoint host tp1
RP/0/RP0/CPU0:5508-1-731(config)#ssh server certificate username ?
  common-name          user common name(CN) from subject name field
  user-principle-name  user principle name(UPN) from subject alternate name
RP/0/RP0/CPU0:5508-1-731(config)#ssh server certificate username</code>
</pre>
</div>

Config guide: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/70x/b-system-security-cg-ncs5500-70x/b-system-security-cg-ncs5500-70x_chapter_011.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/security/70x/b-system-security-cg-ncs5500-70x/b-system-security-cg-ncs5500-70x_chapter_011.html)


## MPLS Segment Routing

This section is split itself in three parts

### IGPs

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/5myoWuSioGY" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

Jose Liste covers a lot of topics in this 30min long video:  

- OSPF LFA/TI-LFA for FlexAlgo

![SRMPLS01.png]({{site.baseurl}}/images/SRMPLS01.png){: .align-center}

Extends the benefits of TI-LFA to Flex-Algo. Automated Topology Independent and guaranteed sub-50ms per-prefix protection optimized per Flex-Algo.  
Flex-Algo will run the shortest path computation for the primary and now also for the back-up path (and with OSPF). The back-up path is computed independantly.

In this example, the flex-algo 128 definition is "min IGP metric and avoid RED affinity". 

![SRMPLS02.png]({{site.baseurl}}/images/SRMPLS02.png){: .align-center}

![SRMPLS03.png]({{site.baseurl}}/images/SRMPLS03.png){: .align-center}

The support is now extended for OSPF at the level it was for ISIS:

![SRMPLS04.png]({{site.baseurl}}/images/SRMPLS04.png){: .align-center}

- Rounding of Min-Delay Values
- OSPF Conditional Advertisement
- ISIS TI-LFA Protection of Unlabeled IPv6 Prefixes
- Inter-Level SRMS Advertisement Propagation

### Performance Monitoring and Traffic-Engineering

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/KkdX7CL7bGU" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

- Named Profile
- SR Policy Delay Measurement with Loopback-mode
- SR Policy Liveless Monitoring
- SR-TE Cumulative-Metric Bound

### Dynamic SR P2MP Policies

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/oG11YKMzxqU" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

- Dynamic SR P2MP policies theory and lab demo

## Optics

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/ybdHKwktP_U" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

## New products

### NCS57B1-6D24 / NCS57B1-5DSE

[https://xrdocs.io/ncs5500/tutorials/introducing-ncs57b1-routers/](https://xrdocs.io/ncs5500/tutorials/introducing-ncs57b1-routers/)

### Chassis commons

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/D57C38pcfyo" frameborder="0" allowfullscreen></iframe>{: .align-center}
.
