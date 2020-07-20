---
published: true
date: '2020-07-18 15:14 +0200'
title: BGP FlowSpec to ACL Script
author: Nicolas Fevrier
excerpt: Introduction and validation of the BGPFS2ACL script
position: top
tags:
  - iosxr
  - script
  - flowspec
  - ncs5500
---
{% include toc icon="table" title="BGPFS2ACL" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

BGP Flowspec is a technology we described in multiple posts and videos:  
- [https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/](https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/)
- [Cisco NCS5500 Flowspec (Principles and Configuration) Part1](https://www.youtube.com/watch?v=dTgh0p9Vyns)
- [Cisco NCS5500 Flowspec (Auto-Mitigation of a Memcached Attack) Part2](https://www.youtube.com/watch?v=iRPob7Ws2v8)

In July 2020, we support BGP Flowspec only on the NCS5500 products equiped with the following ASICs and memories:
- Jericho+ with OP eTCAM
	- NCS55A1-36H-SE-S
    - NCS55A2-MOD-SE-S
    - NC55-36X100G-A-SE
- Jericho2 with OP2 eTCAM
	- NC57-18DD-SE

All these systems and line cards can program the flowspec rules in the datapath, matching packets based on a description and applying actions like drop, rate-limit, remark dscp, redirect to IP or to VRF. But the rest of the portfolio can't support it.   
Nevertheless, it's possible to configure a BGP FS client on all the IOS XR, even if they are not able to program the hardware leveL.  
That's an useful feature because we can develop a program executed on the router itself to convert the flowspec rules received into configuration line. That's exactly what the bgpfs2acl script is doing.  
Routers powered by the Jericho+ with large LPM (NCS55A1-24H, NCS55A1-48Q6H, NCS55A1-24Q6H-SS) are used in peering position and will be perfect candidates for this script.

Thanks to Carl Fredrik Lagerfeldt and Johan Gustawsson who brought first the original idea.

## Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/8pNZJUHlRPk?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

In this interview, Mike explained the basic tools used by the script to operate (ZTP config, etc). Since the day of this interview, Dmitrii Rudnev worked on the script and changed the structure entirely, extending the capabilities significantly.

## The script

### Github

Code is available on Github: [https://github.com/ios-xr/bgpfs2acl](https://github.com/ios-xr/bgpfs2acl).

### Description / Match

The script checks the flowspec rules every 30 seconds.  
In this release, it covers most description options with the following exceptions:  
- no match on DSCP field
- no match on TCP flag
- match on ICMP type and code works but no support of lists or ranges in type and code
- no match on Dont Fragment

### Action

In term of action, it supports drop and redirect-to-IP but not:
- set dscp
- rate limit to X bps
- redirect to VRF

### Support of packet-length ranges

To support the match in packet length (including ranges), it's necessary to enable a specific UDK configuration. We recommend to configure it if you plan to use the script, even before 7.0.1.

<div class="highlighter-rouge">
<pre class="highlight">
<code>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit port-range</code>
</pre>
</div>

Starting up the script, it will check the presence of this config line and warn it will not be able to "translate" rules containing length ranges if it's not enabled.  
If it receives such a rule and the config is not present, it will trigger a message to inform the rule has not been handled.  

Note: enabling this command requires a reload of the line card or chassis.
{: .notice—info}

Note2: the NCS5500 is limited in the number of range IDs, the script is not aware of the limitation and will try to configure as many range than received in the rules.
{: .notice—info}

### Support of fragments

ACL match on fragment-type is only supported on systems with external TCAM because it requires to enable the compression feature. It goes against the main purpose of the script, which is to mimic the flowspec behavior on non-eTCAM routers.  
But matching fragments (without going in deeper level of subtleties like First Fragment, Last Fragment, Is Fragmented, etc) is supported, that why all frag-types described in the flowspec rule will be translated into the same ACL line "fragments".  

## Examples

In the next 9 examples, we demonstrate the capabilities of the script in term of match and actions.

**Cannonball** is the BGP Flowspec Client, receiving the rules and where the script is executed.  
It's an NCS-5501 (non-SE) running IOS XR 7.0.2:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh ver
Cisco IOS XR Software, Version 7.0.2
Copyright (c) 2013-2020 by Cisco Systems, Inc.

Build Information:
 Built By     : ahoang
 Built On     : Fri Mar 13 22:56:17 PDT 2020
 Built Host   : iox-ucs-027
 Workspace    : /auto/srcarchive15/prod/7.0.2/ncs5500/ws
 Version      : 7.0.2
 Location     : /opt/cisco/XR/packages/
 Label        : 7.0.2

cisco NCS-5500 () processor
System uptime is 3 days 13 hours 57 minutes

RP/0/RP0/CPU0:Cannonball#sh platf
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        NCS-5501(Active)           IOS XR RUN        NSHUT
0/RP0/NPU0        Slice                      UP
0/FT0             NCS-1RU-FAN-FW             OPERATIONAL       NSHUT
0/FT1             NCS-1RU-FAN-FW             OPERATIONAL       NSHUT
0/PM0             NCS-1100W-ACFW             OPERATIONAL       NSHUT
0/PM1             NCS-1100W-ACFW             FAILED            NSHUT
RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

**Macrocarpa** is the BGP Flowspec controller injecting/pushing the rules to Cannonball, it runs IOSXR 6.6.3.

### Example1: Destination IP and UDP source port

On the controler:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all CHARGEN
   match destination-address ipv4 7.7.7.7 255.255.255.255
   match protocol udp
   match source-port 19
policy-map type pbr Example1
 class type traffic CHARGEN
  drop
 !
!
flowspec
 address-family ipv4
  service-policy type pbr Example1
!</code>
</pre>
</div>

Before the receiving the BGP FS rule on the client side. The script is enabled and created a bgpfs2acl-ipv4 and applied it on the interface where no ACL was present (Te0/0/0/2). This ACL is empty at the moment with just a permit any any.  
Another ACL test2 exists and is applied to Te0/0/0/1.

<div class="highlighter-rouge">
<pre class="highlight">
<code>interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group test2 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 permit ipv4 any any
!</code>
</pre>
</div>

The BGP FS rule is received on the client. The script kicks in and change the configuration.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball# 
RP/0/RP0/CPU0:Jul 16 13:07:36.861 UTC: config[68949]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000359' to view the changes.

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

We can see how the ACLs have been modified by the script:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list

ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any eq 19 host 7.7.7.7
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

The ACE line 100501 represents the translation of our rule:  
   match destination-address ipv4 7.7.7.7 255.255.255.255  
   match protocol udp  
   match source-port 19  

Note the ACL entries created by the script are always "signaled" by two remarks "FLOWSPEC RULES BEGIN" and "FLOWSPEC RULES END".
{: .notice—info}

Now we verify the behavior of the script when the BGP FS rule is removed:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:06:37.126 UTC: config[67938]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000360' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list

ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
</code>
</pre>
</div>

### Example2: Destination IP and UDP destination port

On the controler:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all SunRPC
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match destination-port 111
 end-class-map
!
policy-map type pbr Example2
 class type traffic SunRPC
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example2
 !
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:09:07.034 UTC: config[67321]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000361' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any host 7.7.7.7 eq sunrpc
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any host 7.7.7.7 eq sunrpc
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
</code>
</pre>
</div>

### Example3: Source Prefix, Destination host, Source UDP Port

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all CHARGEN2
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match source-address ipv4 80.2.1.0 255.255.255.0
 match protocol udp
 match source-port 19
 end-class-map
!
policy-map type pbr Example3
 class type traffic CHARGEN2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example3
 !
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
</code>
</pre>
</div>

### Example4: Destination Host, UDP Source Port Range

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all NETBIOS
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 137-138
 end-class-map
!
policy-map type pbr Example4
 class type traffic NETBIOS
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example4
 !
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:24:07.012 UTC: config[66986]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000365' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
</code>
</pre>
</div>

### Example5: Packet Length Range

Note it's necessary to have the UDK hw-module configured (keep in mind it requires a reload).

On the controller: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all DNS
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 53
 match packet length 768-65535
 end-class-map
!
policy-map type pbr Example5
 class type traffic DNS
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example5
 !
!</code>
</pre>
</div>

Nothing happens on the client.  
Expected since the NCS5500 doesn’t support packet length range larger than 16k.  
We change on the controller a range in the supported scope.


<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all DNS
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 53
 match packet length 768-1600
 end-class-map
!
policy-map type pbr Example5
 class type traffic DNS
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example5
 !
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:33:07.092 UTC: config[67272]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000367' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq domain host 7.7.7.7 packet-length range 768 1600
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any eq domain host 7.7.7.7 packet-length range 768 1600
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#show controllers fia diagshell 0 "diag field Ranges" location 0/0/CPU0

Node ID: 0/0/CPU0
============================================================
| Range Dump                                               |
============================================================
| Qualifier      | Range | Flags           | Min   | Max   |
============================================================
| RangeCheck(26) | 1     | TCP UDP SrcPort | 49152 | 65535 |
============================================================
RP/0/RP0/CPU0:Cannonball#
</code>
</pre>
</div>

### Example6: Multiple Packet-length Ranges

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all NTP2
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 123
 match packet length 1-35 37-45 47-75 77-219 221-65535
 end-class-map
!
policy-map type pbr Example6
 class type traffic NTP2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example6
 !
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100502 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100503 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100504 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100505 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100508 remark FLOWSPEC RULES END
 100509 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Note: the range 221-65535 has been refused but all other ranges have been configured.  

On the controller again we remove one range:  
(on the controller)

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#conf
RP/0/RP0/CPU0:Macrocarpa(config)#class-map type traffic match-all NTP2
RP/0/RP0/CPU0:Macrocarpa(config-cmap)#no  match packet length 1-35 37-45 47-75 77-219 221-65535
RP/0/RP0/CPU0:Macrocarpa(config-cmap)# match packet length 1-35 37-45 47-75 221-65535
RP/0/RP0/CPU0:Macrocarpa(config-cmap)#commit
Thu Jul 16 15:42:10.899 UTC
RP/0/RP0/CPU0:Macrocarpa(config-cmap)#end
RP/0/RP0/CPU0:Macrocarpa#</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100502 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100503 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100504 remark FLOWSPEC RULES END
 100505 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100505 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100507 remark FLOWSPEC RULES END
 100508 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

### Example7: ICMP Type and Code

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all ICMP
match destination-address ipv4 2.2.2.0 255.255.255.0
 match ipv4 icmp-code 2
 match ipv4 icmp-type 2
 end-class-map
!
policy-map type pbr Example7
 class type traffic ICMP
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Example7
!
!</code>
</pre>
</div>

On the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,ICMPType:=2,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list bgpfs2acl-ipv4
Fri Jul 17 15:43:07.863 UTC
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny icmp any 2.2.2.0/24 2 2
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

### Example8: Fragments

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map type pbr Example8a
 class type traffic FRAG1
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example8b
 class type traffic FRAG2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example8c
 class type traffic FRAG3
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
class-map type traffic match-all FRAG1
 match destination-address ipv4 70.2.1.1 255.255.255.255
 match fragment-type  is-fragment
 end-class-map
!
class-map type traffic match-all FRAG2
 match destination-address ipv4 70.2.1.2 255.255.255.255
 match fragment-type  first-fragment
 end-class-map
!
class-map type traffic match-all FRAG3
 match destination-address ipv4 70.2.1.3 255.255.255.255
 match fragment-type  last-fragment
 end-class-map
!
</code>
</pre>
</div>

We have 3 different rules for IsFrag, FirstFrag and LastFrag.

Example8a: is-fragment

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any host 70.2.1.1 fragments
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny ipv4 any host 70.2.1.1 fragments
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Example8b: first-fragment

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any host 70.2.1.2 fragments
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny ipv4 any host 70.2.1.2 fragments
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Example8c: last-fragment

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any host 70.2.1.3 fragments
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 deny ipv4 any host 70.2.1.3 fragments
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

### Example9: Action Redirect-to-IP

In this test, we will test 3 different / overlapping rules with 3 different next-hop addresses, and we will advertise them one by one and see where they appear in the order of operation / ACE numbering.

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all test9a
 match destination-address ipv4 70.2.1.0 255.255.255.0
 end-class-map
!
policy-map type pbr example9a
 class type traffic test9a
  redirect ipv4 nexthop 16.16.16.2
 !
 class type traffic class-default
 !
 end-policy-map
!
class-map type traffic match-all test9b
 match destination-address ipv4 70.2.1.1 255.255.255.255
 end-class-map
!
policy-map type pbr example9b
 class type traffic test9b
  redirect ipv4 nexthop 16.16.16.3
 !
 class type traffic class-default
 !
 end-policy-map
!
class-map type traffic match-all test9c
 match destination-address ipv4 70.0.0.0 255.0.0.0
 end-class-map
!
policy-map type pbr example9c
 class type traffic test9c
  redirect ipv4 nexthop 16.16.16.4
 !
 class type traffic class-default
 !
 end-policy-map
!

RP/0/RP0/CPU0:Macrocarpa(config)#flows
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#address-family ipv4
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#service-policy type pbr example9a
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#commit
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:58:07.139 UTC: config[68582]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000381' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:70.2.1.0/24
    Actions      :Nexthop: 16.16.16.2  (bgp.1)
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Second rule:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#service-policy type pbr example9b
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#commit</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 15:59:07.151 UTC: config[69182]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000382' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:70.2.1.1/32
    Actions      :Nexthop: 16.16.16.3  (bgp.1)
  Flow           :Dest:70.2.1.0/24
    Actions      :Nexthop: 16.16.16.2  (bgp.1)
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 permit ipv4 any host 70.2.1.1 nexthop1 ipv4 16.16.16.3
 100502 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100503 remark FLOWSPEC RULES END
 100504 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 permit ipv4 any host 70.2.1.1 nexthop1 ipv4 16.16.16.3
 100505 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100506 remark FLOWSPEC RULES END
 100507 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Third rule:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#service-policy type pbr example9c
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#commit
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#end
RP/0/RP0/CPU0:Macrocarpa#</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 16 16:00:07.142 UTC: config[66018]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000383' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:70.2.1.1/32
    Actions      :Nexthop: 16.16.16.3  (bgp.1)
  Flow           :Dest:70.2.1.0/24
    Actions      :Nexthop: 16.16.16.2  (bgp.1)
  Flow           :Dest:70.0.0.0/8
    Actions      :Nexthop: 16.16.16.4  (bgp.1)
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list

ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 permit ipv4 any host 70.2.1.1 nexthop1 ipv4 16.16.16.3
 100502 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100503 permit ipv4 any 70.0.0.0/8 nexthop1 ipv4 16.16.16.4
 100504 remark FLOWSPEC RULES END
 100505 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100503 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100504 permit ipv4 any host 70.2.1.1 nexthop1 ipv4 16.16.16.3
 100505 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100506 permit ipv4 any 70.0.0.0/8 nexthop1 ipv4 16.16.16.4
 100507 remark FLOWSPEC RULES END
 100508 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

## Validation / Tests

In this section, we carried out test to validate the script behavior in various conditions.

### Test 01: Starting the script with existing ACL config

**Step 0**

- interface A configured with IPv4 address, shut (down/down), no ACL applied
- interface B configured with IPv4 address, no shut (up/up), no ACL applied
- interface C configured with IPv4 address, no shut (up/up), ACL test2 applied in ingress
- bgp flowspec configured on the client but session down (controller in "shutdown" state)

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh ru int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!
interface TenGigE0/0/0/3
 shutdown
!
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#sh run router bgp
router bgp 65000
 bgp router-id 18.18.18.30
 address-family ipv4 unicast
  network 192.0.3.0/24
  network 192.168.1.2/32
  network 200.1.1.1/32
 !
 address-family ipv4 flowspec
 !
 neighbor 16.16.16.20
  remote-as 65000
  shutdown
  address-family ipv4 unicast
  !
  address-family ipv4 flowspec
   route-policy PASS-ALL in
   route-policy PASS-ALL out
  !
 !
RP/0/RP0/CPU0:Macrocarpa#sh run flows
flowspec
!

RP/0/RP0/CPU0:Macrocarpa#</code>
</pre>
</div>

**Step 1**

- start script
- check interfaces on the client
- check access-lists

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#term mon
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 14:24:40.305 UTC: bash_cmd[67549]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$ docker start bgpfs2acl
bgpfs2acl
[Cannonball:~]$ exit
logout

RP/0/RP0/CPU0:Jul 17 14:24:49.906 UTC: bash_cmd[67549]: %INFRA-INFRA_MSG-5-RUN_LOGOUT : User cisco logged out of shell from vty0
RP/0/RP0/CPU0:Cannonball#

RP/0/RP0/CPU0:Cannonball#sh run int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!
RP/0/RP0/CPU0:Cannonball#acc
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

So, clearly nothing happens.

**Step 2**

- no shut the flowspec on the controller but no rule advertised
- check interfaces on the client,
- check access-lists

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#conf
RP/0/RP0/CPU0:Macrocarpa(config)#router bgp 65000
RP/0/RP0/CPU0:Macrocarpa(config-bgp)# neighbor 16.16.16.20
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#no shut
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#commit
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#end
RP/0/RP0/CPU0:Macrocarpa#</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 14:28:56.185 UTC: bgp[1080]: %ROUTING-BGP-5-ADJCHANGE : neighbor 16.16.16.30 Up (VRF: default) (AS: 65000)
RP/0/RP0/CPU0:Jul 17 14:28:56.186 UTC: bgp[1080]: %ROUTING-BGP-5-NSR_STATE_CHANGE : Changed state to Not NSR-Ready

RP/0/RP0/CPU0:Cannonball#sh bgp ipv4 flows sum
BGP router identifier 16.16.16.20, local AS number 65000
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 42
BGP main routing table version 42
BGP NSR Initial initsync version 2 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker              42         42         42         42          42           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
16.16.16.30       0 65000    1712    1691       42    0    0 00:00:24          0

RP/0/RP0/CPU0:Cannonball#sh run int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress

!
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Here again, the script doesn’t kick in since no BGPFS rule are advertised

**Step 3**

- advertise basic rule on the controller
- client receives BGP FS rule
- client: script creates the ACL “bgpfs2acl” and adds ACE(s) for each match/action with the appropriate remark
- client: script update existing ACL "test2" and add a second with the appropriate remark

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#conf
RP/0/RP0/CPU0:Macrocarpa(config)#flowspec
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)# address-family ipv4
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#  service-policy type pbr Example1
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#commit
RP/0/RP0/CPU0:Macrocarpa(config-flowspec-af)#end
RP/0/RP0/CPU0:Macrocarpa#</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 14:34:57.948 UTC: config[69179]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000399' to view the changes.

RP/0/RP0/CPU0:Cannonball#show configuration commit changes 1000000399
Building configuration...
!! IOS XR Configuration 7.0.2
ipv4 access-list test2
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
interface TenGigE0/0/0/1
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface HundredGigE0/0/1/2
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface HundredGigE0/0/1/4
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface HundredGigE0/0/1/5
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
end

RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#sh run int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq 19 host 7.7.7.7
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

We see that Management and loopback interfaces are not modified.  
Also, interfaces in shutdown state are not modified either.  
ACL "bgpfs2acl-ipv4” is created and applied to interface TenGigE0/0/0/1  
ACL “test2” is modified since it existed and was already applied to TenGigE0/0/0/2.

**Step4**

We stop the rule advertisement from the controller.  
- remove the rule advertisement from the controller
- client: check the impact on the access-lists 

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#conf
RP/0/RP0/CPU0:Macrocarpa(config)#flowspec
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#no address-family ipv4
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#commit
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 14:56:27.929 UTC: config[66774]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000400' to view the changes.

RP/0/RP0/CPU0:Cannonball#int
RP/0/RP0/CPU0:Cannonball#sh run int
Fri Jul 17 14:58:24.079 UTC
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
Fri Jul 17 14:58:26.525 UTC
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

The ACLs “test2” and “bgpfs2acl-ipv4” have been modified but they are still applied to the interfaces.

### Test 02: Ignore rules not supported by the script 

**Step0**

- interface A configured with IPv4 address, shut (down/down), no ACL applied
- interface B configured with IPv4 address, no shut (up/up), bgpfs2acl-ipv4 ACL applied
- interface C configured with IPv4 address, no shut (up/up), ACL test2 applied in ingress
- bgp flowspec session configured and established but no rule advertised

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!
interface TenGigE0/0/0/3
 shutdown

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#sh flows ipv4
Fri Jul 17 15:09:48.070 UTC
RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

**Step1**

- advertise bad rules with multiple unsupported match statements or unsupported actions from the controller 
- check the configuration of the ACL on the client

Controller: Multiple icmp-code in the same rule

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all bflow
 match destination-address ipv4 2.2.2.0 255.255.255.0
 match ipv4 icmp-code 2 3
 match ipv4 icmp-type 2
 end-class-map
!
policy-map type pbr Bad-Rule
class type traffic bflow
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
flowspec
 address-family ipv4
  service-policy type pbr Bad-Rule
!</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,ICMPType:=2,ICMPCode:=2|=3
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#acc
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
Fri Jul 17 16:01:47.933 UTC
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#bash
[Cannonball:~]$ docker logs bgpfs2acl
…
2020-07-17 15:45:23,135 - INFO - Failed to convert flow: Dest:2.2.2.0/24,ICMPType:=2,ICMPCode:=2|=3. Errors: ICMPCode: bgpfs2acl doesn't support icmp ranges: =2|=3
[Cannonball:~]$</code>
</pre>
</div>

Conclusion, create multiple rules. In this example, 2/2 and 2/3.

Other example: Don't Fragement

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Frag:=DF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 16:19:40.247 UTC: bash_cmd[67397]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$ docker logs bgpfs2acl
...
2020-07-17 16:19:23,125 - INFO - Failed to convert flow: Dest:2.2.2.0/24,Frag:=DF. Errors: Frag: Unsupported fragment type value: =DF
[Cannonball:~]$</code>
</pre>
</div>

Other example: TCP Flags

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,TCPFlags:=0x40
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 16:23:35.393 UTC: bash_cmd[69290]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$
[Cannonball:~]$ docker logs bgpfs2acl
…
2020-07-17 16:23:23,142 - INFO - Failed to convert flow: Dest:2.2.2.0/24,TCPFlags:=0x40. Errors: TCPFlags: Unsupported keyword

!</code>
</pre>
</div>

Other example: match DSCP

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,DSCP:=46
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 16:26:01.176 UTC: bash_cmd[66788]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$ docker logs bgpfs2acl
…
2020-07-17 16:25:23,158 - INFO - Failed to convert flow: Dest:2.2.2.0/24,DSCP:=46. Errors: DSCP: Unsupported keyword</code>
</pre>
</div>

Other example: Rate-limiting action

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24
    Actions      :Traffic-rate: 1000 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 16:36:35.751 UTC: bash_cmd[66763]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$ docker logs bgpfs2acl
...
2020-07-17 16:35:23,075 - INFO - Failed to convert flow: Dest:2.2.2.0/24. Errors: action: Usupported action: Traffic-rate: 1000 bps  (bgp.1)</code>
</pre>
</div>

Other example: Set DSCP action

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh flows ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24
    Actions      :DSCP: ef  (bgp.1)
RP/0/RP0/CPU0:Cannonball#bash
RP/0/RP0/CPU0:Jul 17 16:40:01.670 UTC: bash_cmd[68494]: %INFRA-INFRA_MSG-5-RUN_LOGIN : User cisco logged into shell from vty0
[Cannonball:~]$ docker logs bgpfs2acl
…
2020-07-17 16:39:53,141 - INFO - Failed to convert flow: Dest:2.2.2.0/24. Errors: action: Usupported action: DSCP: ef  (bgp.1)
[Cannonball:~]$</code>
</pre>
</div>

### Test 03: Creation of a new IPv4 interface

**Step0**

- interface A configured with IPv4 address, shut (down/down), no ACL applied
- interface B configured with IPv4 address, no shut (up/up), bgpfs2acl-ipv4 ACL applied
- interface C configured with IPv4 address, no shut (up/up), ACL test2 applied in ingress
- bgp flowspec session configured and established but a simple rule is advertised

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#sh run int
interface Loopback0
 ipv4 address 11.11.11.11 255.255.255.255
!
interface MgmtEth0/RP0/CPU0/0
 ipv4 address 10.30.111.177 255.255.255.224
 lldp
  enable
 !
!
interface TenGigE0/0/0/0
 ipv4 address 33.44.77.88 255.255.255.0
 shutdown
!
interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group test2 ingress
!
interface TenGigE0/0/0/3
 shutdown
!</code>
</pre>
</div>

**Step1**

- no shut of interface Te0/0/0/3 but still no IPv4 addresses

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball(config)#interface TenGigE0/0/0/3
RP/0/RP0/CPU0:Cannonball(config-if)#no shut
RP/0/RP0/CPU0:Cannonball(config-if)#commit
LC/0/0/CPU0:Jul 17 16:46:18.959 UTC: ifmgr[259]: %PKT_INFRA-LINK-3-UPDOWN : Interface TenGigE0/0/0/3, changed state to Down
LC/0/0/CPU0:Jul 17 16:46:18.959 UTC: ifmgr[259]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface TenGigE0/0/0/3, changed state to Down
RP/0/RP0/CPU0:Jul 17 16:46:19.733 UTC: config[67444]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'cisco'. Use 'show configuration commit changes 1000000410' to view the changes.
RP/0/RP0/CPU0:Cannonball(config-if)#
RP/0/RP0/CPU0:Cannonball(config-if)#end
RP/0/RP0/CPU0:Jul 17 16:46:21.568 UTC: config[67444]: %MGBL-SYS-5-CONFIG_I : Configured from console by cisco on vty0 (10.209.200.69)
RP/0/RP0/CPU0:Cannonball#sh run int TenGigE0/0/0/3
% No such configuration item(s)

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

As expected no modification has been applied, the script needs an L3 interface and for instance an IPv4 configured.

**Step2**

- we add IPv4 address to this interface

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#conf
RP/0/RP0/CPU0:Cannonball(config)#int TenGigE0/0/0/3
RP/0/RP0/CPU0:Cannonball(config-if)#ipv4 add 11.44.22.33/24
RP/0/RP0/CPU0:Cannonball(config-if)#commit
RP/0/RP0/CPU0:Jul 17 16:47:41.512 UTC: config[68122]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'cisco'. Use 'show configuration commit changes 1000000411' to view the changes.
RP/0/RP0/CPU0:Cannonball(config-if)#end
RP/0/RP0/CPU0:Jul 17 16:47:44.599 UTC: config[68122]: %MGBL-SYS-5-CONFIG_I : Configured from console by cisco on vty0 (10.209.200.69)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#sh run int RP/0/RP0/CPU0:Jul 17 16:47:58.056 UTC: config[68408]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000412' to view the changes.
RP/0/RP0/CPU0:Cannonball#sh run int TenGigE0/0/0/3
interface TenGigE0/0/0/3
 ipv4 address 11.44.22.33 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

As soon as interface is up/up AND an address is configured, the script will add the bgpfs2acl-ipv4 ACL in ingress

**Step3**

- We use an existing ACL "test3" no applied yet
- We replace the bgpfs2acl-ipv4 ACL on this interface by the "test3" one
- We verify the modification of the "test3" ACL with the addition of the flowspec entries

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball(config-ipv4-acl)#exit
RP/0/RP0/CPU0:Cannonball(config)#do sh run ipv4 access-list test3
 10 permit ipv4 1.2.3.0 0.0.0.24 any
 20 deny icmp any any
 30 permit ipv4 host 3.4.5.6 any
!

RP/0/RP0/CPU0:Cannonball(config)#do sh run int te 0/0/0/3
interface TenGigE0/0/0/3
 ipv4 address 11.44.22.33 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!

RP/0/RP0/CPU0:Cannonball(config)#interface TenGigE0/0/0/3
RP/0/RP0/CPU0:Cannonball(config-if)# ipv4 access-group test3 ingress
RP/0/RP0/CPU0:Cannonball(config-if)#commit
RP/0/RP0/CPU0:Jul 17 16:53:35.668 UTC: config[69624]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'cisco'. Use 'show configuration commit changes 1000000414' to view the changes.
RP/0/RP0/CPU0:Cannonball(config-if)#end
RP/0/RP0/CPU0:Jul 17 16:53:37.711 UTC: config[69624]: %MGBL-SYS-5-CONFIG_I : Configured from console by cisco on vty0 (10.209.200.69)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 16:53:58.083 UTC: config[67380]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000415' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run int te 0/0/0/3
interface TenGigE0/0/0/3
 ipv4 address 11.44.22.33 255.255.255.0
 ipv4 access-group test3 ingress
!

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list test3
ipv4 access-list test3
 10 permit ipv4 1.2.3.0 0.0.0.24 any
 20 deny icmp any any
 30 permit ipv4 host 3.4.5.6 any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

The script detected the association of this ACL test3 to an interface and changed the content, adding the translated BGPFS rule

### Test 04: BGP FS session lost

- from the controller, we shut down the session
- on the client, all rules are deleted
- script will remove all ACEs

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa(config)#router bgp 65000
RP/0/RP0/CPU0:Macrocarpa(config-bgp)#
RP/0/RP0/CPU0:Macrocarpa(config-bgp)# neighbor 16.16.16.20
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#shut
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#commit
RP/0/RP0/CPU0:Macrocarpa(config-bgp-nbr)#</code>
</pre>
</div>

Client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 17:19:21.750 UTC: bgp[1080]: %ROUTING-BGP-5-ADJCHANGE : neighbor 16.16.16.30 Down - BGP Notification received, administrative shutdown (VRF: default) (AS: 65000)
RP/0/RP0/CPU0:Jul 17 17:19:21.750 UTC: bgp[1080]: %ROUTING-BGP-5-NSR_STATE_CHANGE : Changed state to NSR-Ready
RP/0/RP0/CPU0:Jul 17 17:19:58.044 UTC: config[66780]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000416' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh flows ipv4
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 permit ipv4 any any
!
ipv4 access-list test3
 10 permit ipv4 1.2.3.0 0.0.0.24 any
 20 deny icmp any any
 30 permit ipv4 host 3.4.5.6 any
 100500 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

And we restore the session:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 17:22:06.632 UTC: bgp[1080]: %ROUTING-BGP-5-ADJCHANGE : neighbor 16.16.16.30 Up (VRF: default) (AS: 65000)
RP/0/RP0/CPU0:Jul 17 17:22:06.632 UTC: bgp[1080]: %ROUTING-BGP-5-NSR_STATE_CHANGE : Changed state to Not NSR-Ready
RP/0/RP0/CPU0:Jul 17 17:22:28.187 UTC: config[68049]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000417' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list test2
 10 deny ipv4 any host 1.2.3.4
 20 permit ipv4 any host 2.3.4.5
 30 remark end test2
 40 deny ipv4 any any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list test3
 10 permit ipv4 1.2.3.0 0.0.0.24 any
 20 deny icmp any any
 30 permit ipv4 host 3.4.5.6 any
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny ipv4 any 2.2.2.0/24
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

### Test 05: manipulation of the created ACL

The script will modify existing ACLs if they are already applied but will not change ACLs configured but not associated to interfaces.  
Also, it will create a specific ACL for interfaces up/up with an IPv4 address configured. This ACL bgpfs2acl-ipv4 can be also edited by user as long as nothing if modified in between the two remarks.

- a rule is advertised and translated in the client
- check existing bgpfs2acl-ipv4 access-list
- add ACE / entries in this ACL
- withdraw the rule advertisement from the controller
- check the ACL bgpfs2acl-ipv4

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list bgpfs2acl-ipv4
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

We add a line in this ACL

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#conf
RP/0/RP0/CPU0:Cannonball(config)#ipv4 access-list bgpfs2acl-ipv4
RP/0/RP0/CPU0:Cannonball(config-ipv4-acl)#10 permit udp any 1.2.3.0/24 eq 80
RP/0/RP0/CPU0:Cannonball(config-ipv4-acl)#commit
RP/0/RP0/CPU0:Cannonball(config-ipv4-acl)#end
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list bgpfs2acl-ipv4
ipv4 access-list bgpfs2acl-ipv4
 10 permit udp any 1.2.3.0/24 eq 80
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

We stop the advertisement on the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Macrocarpa#conf
RP/0/RP0/CPU0:Macrocarpa(config)#flowspec
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#no address-family ipv4
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#commit
Mon Jul 20 09:08:18.309 UTC
RP/0/RP0/CPU0:Macrocarpa(config-flowspec)#</code>
</pre>
</div>

On the client, the ACL bgpfs2acl-ipv4 only contains the entry we manually contain and (it's important) the permit any any at the end.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 20 09:08:32.835 UTC: config[68089]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000449' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list bgpfs2acl-ipv4
ipv4 access-list bgpfs2acl-ipv4
 10 permit udp any 1.2.3.0/24 eq 80
 100500 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Finally, we re-advertise the rule from the controller and check the ACL one last time on the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 20 09:14:02.886 UTC: config[66542]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000450' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-list bgpfs2acl-ipv4
ipv4 access-list bgpfs2acl-ipv4
 10 permit udp any 1.2.3.0/24 eq 80
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>


### Test 06: add up multiple FS rules

We will apply examples 1 to 9, one by one.

Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map type pbr Example1
 class type traffic CHARGEN
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example2
 class type traffic SunRPC
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example3
 class type traffic CHARGEN2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example4
 class type traffic NETBIOS
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example5
 class type traffic DNS
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example6
 class type traffic NTP2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example7
 class type traffic ICMP
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example8a
 class type traffic FRAG1
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example8b
 class type traffic FRAG2
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr Example8c
 class type traffic FRAG3
  drop
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr example9a
 class type traffic test9a
  redirect ipv4 nexthop 16.16.16.2
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr example9b
 class type traffic test9b
  redirect ipv4 nexthop 16.16.16.3
 !
 class type traffic class-default
 !
 end-policy-map
!
policy-map type pbr example9c
 class type traffic test9c
  redirect ipv4 nexthop 16.16.16.4
 !
 class type traffic class-default
 !
 end-policy-map
!

class-map type traffic match-all DNS
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 53
 match packet length 768-65535 768-1601
 end-class-map
!
class-map type traffic match-all FRAG
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match fragment-type  is-fragment
 end-class-map
!
class-map type traffic match-all ICMP
 match destination-address ipv4 70.2.1.1 255.255.255.255
 match ipv4 icmp-type 3
 match ipv4 icmp-code 2
 end-class-map
!
class-map type traffic match-all NTP2
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 123
 match packet length 1-35 37-45 47-75 77-219 221-65535
 end-class-map
!
class-map type traffic match-all FRAG1
 match destination-address ipv4 70.2.1.1 255.255.255.255
 match fragment-type  is-fragment
 end-class-map
!
class-map type traffic match-all FRAG2
 match destination-address ipv4 70.2.1.2 255.255.255.255
 match fragment-type  first-fragment
 end-class-map
!
class-map type traffic match-all FRAG3
 match destination-address ipv4 70.2.1.3 255.255.255.255
 match fragment-type  last-fragment
 end-class-map
!
class-map type traffic match-all SunRPC
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match destination-port 111
 end-class-map
!
class-map type traffic match-all test9a
 match destination-address ipv4 70.2.1.0 255.255.255.0
 end-class-map
!
class-map type traffic match-all test9b
 match destination-address ipv4 70.2.1.1 255.255.255.255
 end-class-map
!
class-map type traffic match-all test9c
 match destination-address ipv4 70.0.0.0 255.0.0.0
 end-class-map
!
class-map type traffic match-all CHARGEN
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 19
 end-class-map
!
class-map type traffic match-all NETBIOS
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match protocol udp
 match source-port 137-138
 end-class-map
!
class-map type traffic match-all CHARGEN2
 match destination-address ipv4 7.7.7.7 255.255.255.255
 match source-address ipv4 80.2.1.0 255.255.255.0
 match protocol udp
 match source-port 19
 end-class-map
!
</code>
</pre>
</div>

On the client, we check the received rules one by one and we verify the ACL created.

First rule, example1

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 18:02:58.125 UTC: config[67395]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000433' to view the changes.
fl
RP/0/RP0/CPU0:Cannonball#show flows ipv4
Fri Jul 17 18:03:00.691 UTC

AFI: IPv4
  <mark>Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 <mark>100501 deny udp any eq 19 host 7.7.7.7</mark>
 100502 remark FLOWSPEC RULES END
 100503 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Let's add the second rule:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  <mark>Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 17 18:03:58.243 UTC: config[68062]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000434' to view the changes.

RP/0/RP0/CPU0:Cannonball#acc
RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l

ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 <mark>100501 deny udp any host 7.7.7.7 eq sunrpc</mark>
 100502 deny udp any eq 19 host 7.7.7.7
 100503 remark FLOWSPEC RULES END
 100504 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

The [RFC5575 Section 5.1](https://tools.ietf.org/html/rfc5575#page-13) defines the "Order of Traffic Filtering Rules" which permits to define an order of precedence between two or more rules.  
" If the types differ, the rule with lowest numeric type value has higher precedence (and thus will match before)". That's why a "Type 1 - Destination Prefix" is applied before a "Type 2 - Source Prefix" and it's indeed reflected in the "show flowspec ipv4" output.  
The script uses this output to define the order of the ACEs it creates.

Third example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  <mark>Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#fl RP/0/RP0/CPU0:Jul 17 18:04:58.161 UTC: config[68662]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000435' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 <mark>100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7</mark>
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any eq 19 host 7.7.7.7
 100504 remark FLOWSPEC RULES END
 100505 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Example 4:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4
Fri Jul 17 18:05:15.319 UTC

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:05:28.192 UTC: config[69101]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000436' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 <mark>100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7</mark>
 100504 deny udp any eq 19 host 7.7.7.7
 100505 remark FLOWSPEC RULES END
 100506 permit ipv4 any any
!

RP/0/RP0/CPU0:Cannonball#</code>
</pre>
</div>

Example 5:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:06:28.106 UTC: config[65605]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000437' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 <mark>100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601</mark>
 100506 remark FLOWSPEC RULES END
 100507 permit ipv4 any any
</code>
</pre>
</div>

Example 6:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:07:28.275 UTC: config[66206]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000438' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 <mark>100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35</mark>
 <mark>100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45</mark>
 <mark>100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75</mark>
 <mark>100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219</mark>
 100510 remark FLOWSPEC RULES END
 100511 permit ipv4 any any
</code>
</pre>
</div>

Example 7:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>

RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:07:58.230 UTC: config[66597]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000439' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 <mark>100510 deny icmp any host 70.2.1.1 protocol-unreachable</mark>
 100511 remark FLOWSPEC RULES END
 100512 permit ipv4 any any
</code>
</pre>
</div>

Example 8:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.1/32,Frag:=IsF</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:09:28.187 UTC: config[67422]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000440' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100510 deny icmp any host 70.2.1.1 protocol-unreachable
 <mark>100511 deny ipv4 any host 70.2.1.1 fragments</mark>
 100512 remark FLOWSPEC RULES END
 100513 permit ipv4 any any</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,Frag:=IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.2/32,Frag:=FF</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:10:28.259 UTC: config[68054]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000441' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100510 deny icmp any host 70.2.1.1 protocol-unreachable
 100511 deny ipv4 any host 70.2.1.1 fragments
 <mark>100512 deny ipv4 any host 70.2.1.2 fragments</mark>
 100513 remark FLOWSPEC RULES END
 100514 permit ipv4 any any
</code>
</pre>
</div>

The Three different rules are translated with "fragments" as explain in the beginning of the post.

Example9:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,Frag:=IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.2/32,Frag:=FF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.3/32,Frag:=LF</mark>
    <mark>Actions      :Traffic-rate: 0 bps  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:11:28.245 UTC: config[68658]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000442' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100510 deny icmp any host 70.2.1.1 protocol-unreachable
 100511 deny ipv4 any host 70.2.1.1 fragments
 100512 deny ipv4 any host 70.2.1.2 fragments
 <mark>100513 deny ipv4 any host 70.2.1.3 fragments</mark>
 100514 remark FLOWSPEC RULES END
 100515 permit ipv4 any any
</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,Frag:=IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.2/32,Frag:=FF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.3/32,Frag:=LF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.0/24</mark>
    <mark>Actions      :Nexthop: 16.16.16.2  (bgp.1)</mark>
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:12:58.245 UTC: config[69474]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000443' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100510 deny icmp any host 70.2.1.1 protocol-unreachable
 100511 deny ipv4 any host 70.2.1.1 fragments
 100512 deny ipv4 any host 70.2.1.2 fragments
 100513 deny ipv4 any host 70.2.1.3 fragments
 <mark>100514 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2</mark>
 100515 remark FLOWSPEC RULES END
 100516 permit ipv4 any any</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#show flows ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Source:80.2.1.0/24,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,DPort:=111
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:>=137&<=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768&<=65535|>=768&<=1601
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123,Length:>=1&<=35|>=37&<=45|>=47&<=75|>=77&<=219|>=221&<=65535
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,ICMPType:=3,ICMPCode:=2
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.1/32,Frag:=IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  <mark>Flow           :Dest:70.2.1.1/32</mark>
    <mark>Actions      :Nexthop: 16.16.16.3  (bgp.1)</mark>
  Flow           :Dest:70.2.1.2/32,Frag:=FF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.3/32,Frag:=LF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:70.2.1.0/24
    Actions      :Nexthop: 16.16.16.2  (bgp.1)
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Cannonball#
RP/0/RP0/CPU0:Jul 17 18:14:58.169 UTC: config[66405]: %MGBL-CONFIG-6-DB_COMMIT : Configuration committed by user 'ZTP'. Use 'show configuration commit changes 1000000444' to view the changes.

RP/0/RP0/CPU0:Cannonball#sh run ipv4 access-l
ipv4 access-list bgpfs2acl-ipv4
 100500 remark FLOWSPEC RULES BEGIN. Do not add statements below this. Added automatically.
 100501 deny udp 80.2.1.0/24 eq 19 host 7.7.7.7
 100502 deny udp any host 7.7.7.7 eq sunrpc
 100503 deny udp any range netbios-ns netbios-dgm host 7.7.7.7
 100504 deny udp any eq 19 host 7.7.7.7
 100505 deny udp any eq domain host 7.7.7.7 packet-length range 768 1601
 100506 deny udp any eq ntp host 7.7.7.7 packet-length range 1 35
 100507 deny udp any eq ntp host 7.7.7.7 packet-length range 37 45
 100508 deny udp any eq ntp host 7.7.7.7 packet-length range 47 75
 100509 deny udp any eq ntp host 7.7.7.7 packet-length range 77 219
 100510 deny icmp any host 70.2.1.1 protocol-unreachable
 100511 deny ipv4 any host 70.2.1.1 fragments
 <mark>100512 permit ipv4 any host 70.2.1.1 nexthop1 ipv4 16.16.16.3</mark>
 100513 deny ipv4 any host 70.2.1.2 fragments
 100514 deny ipv4 any host 70.2.1.3 fragments
 100515 permit ipv4 any 70.2.1.0/24 nexthop1 ipv4 16.16.16.2
 100516 remark FLOWSPEC RULES END
 100517 permit ipv4 any any
!</code>
</pre>
</div>

## Misc

The script triggers logging messages related to EDM callback function from sysdb

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Cannonball#RP/0/RP0/CPU0:Jul 20 09:38:56.696 UTC: nvgen[69429]: %MGBL-CONFIG_HIST_UPDATE-3-SYSDB_GET : Error 'sysdb' detected the 'warning' condition 'A verifier or EDM callback function returned: 'not found'' getting host address from  sysdb
RP/0/RP0/CPU0:Jul 20 09:38:57.864 UTC: nvgen[69500]: %MGBL-CONFIG_HIST_UPDATE-3-SYSDB_GET : Error 'sysdb' detected the 'warning' condition 'A verifier or EDM callback function returned: 'not found'' getting host address from  sysdb
RP/0/RP0/CPU0:Jul 20 09:39:26.646 UTC: nvgen[65542]: %MGBL-CONFIG_HIST_UPDATE-3-SYSDB_GET : Error 'sysdb' detected the 'warning' condition 'A verifier or EDM callback function returned: 'not found'' getting host address from  sysdb
RP/0/RP0/CPU0:Jul 20 09:39:27.809 UTC: nvgen[65620]: %MGBL-CONFIG_HIST_UPDATE-3-SYSDB_GET : Error 'sysdb' detected the 'warning' condition 'A verifier or EDM callback function returned: 'not found'' getting host address from  sysdb
</code>
</pre>
</div>

Until this is fixed from the script, you can safely ignore them and the easiest way is to suppress them with the following configuration.

<div class="highlighter-rouge">
<pre class="highlight">
<code>logging suppress rule EDM
 alarm MGBL CONFIG_HIST_UPDATE SYSDB_GET
!
logging suppress apply rule EDM
 all-of-router
!</code>
</pre>
</div>

## Conclusion

Kudos to Mike ([github](https://github.com/Maikor)) for the initial framework and to Dmitrii ([github](https://github.com/rudneff) / linkedin) for the great work delivered during the last weeks on this program.

Other features are "work in progress", particularly a syslog module providing information at the user level (and not only the script logs) of the different actions, errors, etc.  
We really hope this script will help the community, give it a try and provide us your feedback.
