---
published: true
date: '2021-09-09 23:04 +0200'
title: IOS XR 7.4.1 Innovations in NCS5500/NCS5700/NCS500 Platforms
author: Nicolas Fevrier
excerpt: New features and hardware supported in IOS XR 7.4.1
position: top
---
{% include toc icon="table" title="XR 7.4.1 New Features and Hardware for DNX Platforms" %} 


<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/qvczWr2Vu_U" frameborder="0" allowfullscreen></iframe>{: .align-center}
.

## Introduction

IOS XR 7.4.1 has been published in August 2021 and is an ED version for many XR platforms including:  
- NCS5500
- NCS5700
- NCS540
- NCS560

Release notes:
- NCS540: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/74x/b-release-notes-ncs540-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5xx/release-notes/74x/b-release-notes-ncs540-r741.html)  
- NCS560: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/74x/b-release-notes-ncs560-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs560/release-notes/74x/b-release-notes-ncs560-r741.html)
- NCS5500: [https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/74x/release/notes/b-release-notes-ncs5500-r741.html](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/general/74x/release/notes/b-release-notes-ncs5500-r741.html)

Software download for NCS5500:  
[https://software.cisco.com/download/home/286291132/type](https://software.cisco.com/download/home/286291132/type)

![download.png]({{site.baseurl}}/images/download.png){: .align-center}

Note: two flavors of IOSXR are now available for NCS5500/NCS5700 platforms

## Software Images

### XR 64-bit

In this section of the download center, you'll find software images for all platforms, fixed or modular running IOS XR 64-bit, with the particular exception of the NCS-57B1 platforms.

![5500-soft.png]({{site.baseurl}}/images/5500-soft.png){: .align-center}

### XR7

In current release, one single platform type in the NCS5500/NCS5700 series is running the new version of IOS XR named "XR7" ("LNT" in the show commands). That's the NCS57B1-6D24 and NCS57B1-5DSE. And image can be found in this section of the download center (under 5700):

![5700-soft.png]({{site.baseurl}}/images/5700-soft.png){: .align-center}

Note: Even the latest platform NCS57C3-MOD is running XR 64-bit and not XR7.
{: .notice--info}

Please refer to the release notes and installation guide to get details on the upgrade process.

## New Features in IOS XR 7.4.1

We asked a few colleagues to help documenting the new improvements brought by the 7.3.1 version. It will be split in software innovations on one side (Segment Routing, SRv6, EVPN, Multicast, QoS, Security, ...) and new supported hardware (chassis, power supply, new line cards and new fixed-form-factor products).

### ACL

Tejas Lad demonstrates the improvements brought by J2 ASIC for egress ACL and statistics.  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/NJHSheNfDZw" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

With Jericho2 platforms, a series of limitations present in Jericho/Jericho+ have been lifted, for example:  
- We don't need to use a specific hardware module profile configuration to allocate counters for ACL permit packets. "hw-module profile stats acl-permit" is not used anymore.  
- In Jericho and Qumran-MX platforms with NL12k eTCAM, it could be necessary to recarve the resource with "hw-module profile  tcam acl-prefix percent <0-100>". It's all dynamic on Jericho+ with OP and Jericho2 with OP2 eTCAMs, no config is needed
- We don't recycle packets for ACLv6 in egress direction like it was the case in Jericho and Jericho+ platforms, the packet treatment with J2 is done in one pass.
- We support egress ACLv6 on BVI interfaces, something not supported on J/J+ platforms.

Tejas also published another article here with plenty of details:  
[https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/](https://xrdocs.io/ncs5500/tutorials/access-list-enhancements-on-ncs5500-j2-based-platforms/)

In IOS XR 7.4.1, we complete this list of improvements with more fields.

| Match Parameter | IPv4 support pre-741 | IPv4 support in 741 | IPv6 support pre-741 | IPv6 support in 741 |
|:---:|:---:|:---:|:---:|:---:|
| Source Address | Yes | Yes | Yes | Yes |
| Destination Address | Yes | Yes | Yes | Yes |
| Source Port | Yes | Yes | Yes | Yes |
| Destination Port | Yes | Yes | Yes | Yes |
| Protocol/ Next Header | Yes | Yes | Yes | Yes |
| Precedence/DSCP | Yes | Yes | No | Yes |
| Packet Length | No | Yes | No | Yes |
| TCP control Flags | Yes | Yes | No | Yes |
| Fragment bit | Yes | Yes | No | Yes |
| Extended Header | N/A | N/A | No | Yes |
| Destination System port | Yes | Yes | Yes | Yes |

### Security

Rakesh Kandula presented new security feature: Chip Guard on NCS540.  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/2fwIZRRJPSw" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

Let's first describe two essential components into every IOS XR router:  
- TAm: the Trust Anchor module chip integrated in all recent device
- Cisco Secure Boot: the chain ensuring the integrity of the entire boot process

![Secure-boot-process.png]({{site.baseurl}}/images/Secure-boot-process.png){: .align-center}

Also these components are not enough if the hardware itself has been replaced by malware infected parts (CPU or NPU). And that's the purpose of this new feature: Chip Guard introduced now in our access platforms: NCS540.

Chip Guard feature is triggered during the BIOS part of the boot sequence.  

![Manufacturing-diag-process.png]({{site.baseurl}}/images/Manufacturing-diag-process.png){: .align-center}

![ImprintDB.png]({{site.baseurl}}/images/ImprintDB.png){: .align-center}
During manufacturing:
- the SHA-256 hash of the Electronic Chip ID (ECID) of the CPU and NPU are calculated
- These hashes are then programmed inside the TAm chip
- The programmed hash values form the ImprintDB inside the TAm chip
- The ImprintDB cannot be modified during runtime

When the router boots:  

![Step1.png]({{site.baseurl}}/images/Step1.png){: .align-center}

- BIOS reads the ECID of the chips and computes their hashes
- Each of the hashes is then extended into a PCR inside TAm chip
- These set of observed hashes forms the ObserveDB

![Step2.png]({{site.baseurl}}/images/Step2.png){: .align-center}

- BIOS fetches the factory programmed hash values from imprintDB
- The hash values are compared with the ObserveDB generated in the previous step
- BIOS continues with boot process if and only if the hashes match

### QoS

Paban Sarma introduces PPS-based policer for DNX platforms (NCS500 / NCS5500).  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/xUFR_DVV1bo" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

Before the introduction of this feature, we used absolute values in bps for policer rate and burst in bytes or unit of time:

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map police-bps
 class class-default
  police rate 10 mpbs burst 12 kbytes
  !
 !
 end-policy-map
!</code>
</pre>
</div>

Or we can use relative units (percent):

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map police-percent
 class class-default
  police rate percent 1 burst 2 ms
  !
 !
 end-policy-map
!</code>
</pre>
</div>

Starting in IOS XR 7.4.1, we can use absolute units of PPS and burst expressed in number of packets.

<div class="highlighter-rouge">
<pre class="highlight">
<code>policy-map police-pps
 class class-default
  police rate 1000 pps burst 2000 packets
  !
 !
 end-policy-map
!

RP/0/RP0/CPU0:NCS-55A1-24Q6H-2#show  qos interface  tenGigE 0/0/0/1.1 input

NOTE:- Configured values are displayed within parentheses
Interface TenGigE0/0/0/1.1 ifh 0x8022  -- input policy
NPU Id:                        0
Total number of classes:       1
Interface Bandwidth:           10000000 kbps
Policy Name:                   police-pps
SPI Id:                        0x0
Accounting Type:               Layer2 (Include Layer 2 encapsulation and above)
------------------------------------------------------------------------------
Level1 Class                             =   class-default

Policer Bucket ID                        =   0x100
Policer Stats Handle                     =   0x0
Policer committed rate                   =   998 kbps (<mark>1000 packets/sec</mark>)
Policer conform burst                    =   256000 bytes (<mark>2000 packets</mark>)</code>
</pre>
</div>

With this approach, the policer behaves regardless of the size of the packets handled by the policy.  
Min and max configurable values are 100 pps and 66,000,000 pps respectively.  
Burst size to be defined in packets PPS and BPS may coexist in a Flat Policy. PPS and BPS can’t coexist in an Hierarchical policy.

### BFD

Tejas Lad presents the BoB-BLB co-existence benefits.  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/60Kuy1NY6JA" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

In IOS XR 7.4.1, we introduce the action of both BoB and BLB simultaneously on the same bundle for faster convergence. All technical details can be found in this dedicated article:  
[https://xrdocs.io/ncs5500/tutorials/coexistence-between-bfd-over-bundle-and-bfd-over-logical-bundle/](https://xrdocs.io/ncs5500/tutorials/coexistence-between-bfd-over-bundle-and-bfd-over-logical-bundle/)

### Scale improvements

Nicolas Fevrier presents the scale improvements directly available in J2 native mode for BGP Flowspec, ARP scale and ECMP FEC.  
Coming soon

### Segment Routing

Jose Liste describes in details the new features introduced in SR MPLS world.  
Coming soon

### Misc

Coming soon

## New hardware in IOS XR 7.4.1

### Introducing NCS57C3-MOD

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/ARKLok7dj-w" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

[https://www.youtube.com/watch?v=ARKLok7dj-w](https://www.youtube.com/watch?v=ARKLok7dj-w)

### Deeper dive in Cisco NCS57C3

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/MV2hNv4xn6Q" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

[https://www.youtube.com/watch?v=MV2hNv4xn6Q](https://www.youtube.com/watch?v=MV2hNv4xn6Q)

### Introducing the new QSFP-DD MPA on NCS5500 Routers

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/6Ksv8oqhBk0" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

[https://www.youtube.com/watch?v=6Ksv8oqhBk0](https://www.youtube.com/watch?v=6Ksv8oqhBk0)

### Newest Cisco Router Unboxed (NCS57C3-MOD Installation)

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/sma8sQwSbfk" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

[https://www.youtube.com/watch?v=sma8sQwSbfk](https://www.youtube.com/watch?v=sma8sQwSbfk)

### New NCS560 IMA and 4x10G support

Paban Sarma introduces the new IMA and the partial 4x10G mode for NCS560:

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/NUEAc8gPP-0" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

IOS XR 7.4.1 is used as the vehicule to introduce the support of a new IMA for NCS560 series (4-slot or 7-slot): N560-IMA-N560-IMA-8Q/4L

![new-IMA.png]({{site.baseurl}}/images/new-IMA.png){: .align-center}

Cisco NCS 560 Series Routers Interface Modules Data Sheet:  
[https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-560-series-routers/datasheet-c78-740295.html](https://www.cisco.com/c/en/us/products/collateral/routers/network-convergence-system-560-series-routers/datasheet-c78-740295.html)

8 ports split in two "QUADs" (groups of 4 contiguous ports):
- QUAD-1: port 0-3
- QUAD-2: port 4-7

You can configure these QUAD in a specific mode (by default, it's 25G) and the four ports of the group will be configured in the same mode:  
- all four ports 10GE
- all four ports 25GE
- one port on two at 50G

So, it's possible to use these ports in various combinations:  
- 8x 10G
- 4x 10G + 4x 25G (or vice versa)
- 4x 10G + 2x 50G (or vice versa)
- 4x 25G + 2x 50G (or vice versa)
- 8x 25G
- 4x 50G

Note1: that configuring a port in 50G mode is only supported in the even-numbered port and it disables the N+1 port.
{: .notice--info}

Note2: in 10G mode, we don't support 1G insertion either
{: .notice--info}

In this example below, we have the IMA inserted in slot 7 and slot 9 and no QUAD configuration, all the ports are 25GE by default.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:N560-7#show version
Wed Aug 11 05:36:26.011 UTC
Cisco IOS XR Software, Version 7.4.1
Copyright (c) 2013-2021 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Thu Feb 25 19:40:08 PST 2021
 Built Host   : iox-ucs-024
 Workspace    : /auto/srcarchive17/prod/7.4.1/ncs560/ws
 Version      : 7.4.1
 Location     : /opt/cisco/XR/packages/
 Label        : 7.4.1

cisco NCS-560 () processor
System uptime is 0 weeks 4 days 12 hours 44 minutes

RP/0/RP1/CPU0:N560-7#show platform 
Wed Aug 11 05:39:29.011 UTC
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/1/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/2/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/4/CPU0          A900-IMA8Z                 OPERATIONAL       NSHUT
0/5/CPU0          A900-IMA8Z                 OPERATIONAL       NSHUT
0/7/CPU0          N560-IMA-8Q/4L             OPERATIONAL       NSHUT
0/9/CPU0          N560-IMA-8Q/4L             OPERATIONAL       NSHUT
0/10/CPU0         A900-IMA8Z                 OPERATIONAL       NSHUT
0/11/CPU0         A900-IMA8Z-L               OPERATIONAL       NSHUT
0/12/CPU0         A900-IMA8Z                 OPERATIONAL       NSHUT
0/13/CPU0         A900-IMA8Z-L               OPERATIONAL       NSHUT
0/RP0/CPU0        N560-RSP4-E(Standby)       IOS XR RUN        NSHUT
0/RP1/CPU0        N560-RSP4-E(Active)        IOS XR RUN        NSHUT
0/FT0/CPU0        N560-FAN-H                 OPERATIONAL       NSHUT
0/PM2/CPU0        A900-PWR1200-A             OPERATIONAL       NSHUT
RP/0/RP1/CPU0:N560-7#
RP/0/RP1/CPU0:N560-7#show ipv4 int brief | inc 0/9/0/
Wed Aug 11 05:42:29.011 UTC
TwentyFiveGigE0/9/0/0          unassigned      Up              Up       default 
TwentyFiveGigE0/9/0/1          120.0.1.1       Up              Up       vpn1    
TwentyFiveGigE0/9/0/2          120.0.5.1       Up              Up       vpn5    
TwentyFiveGigE0/9/0/3          120.0.5.2       Up              Up       vpn6    
TwentyFiveGigE0/9/0/4          120.0.9.2       Up              Up       vpn10   
TwentyFiveGigE0/9/0/5          120.0.9.1       Up              Up       vpn9    
TwentyFiveGigE0/9/0/6          120.0.1.2       Up              Up       vpn2    
TwentyFiveGigE0/9/0/7          120.0.0.1       Up              Up       vpn1</code>
</pre>
</div>

If you configure the QUAD 1 in slot 9 to 10GE, we'll have the first 4 ports in TenGigE and the last 4 ports in TwentyFiveGigE:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:N560-7#conf
Wed Aug 11 05:44:29.011 UTC
RP/0/RP0/CPU0:ios(config)#hw-module quad ?                
  1-2  configure quad properties
RP/0/RP0/CPU0:ios(config)#hw-module quad 1 slot ?
  0-15  configure slot properties
RP/0/RP0/CPU0:ios(config)#hw-module quad 1 slot 9 ?
  mode  select mode 10g or 25g or 50g for a quad(group of 4 ports).
  cr  
RP/0/RP0/CPU0:ios(config)#hw-module quad 1 slot 9 mode 10g ?
  cr  
RP/0/RP0/CPU0:ios(config)#hw-module quad 1 slot 9 mode 10g 
RP/0/RP0/CPU0:ios(config)#commit
RP/0/RP0/CPU0:ios(config)#exit
RP/0/RP1/CPU0:N560-7#
RP/0/RP1/CPU0:N560-7#show ipv4 int brief | inc 0/9/0/ 
Wed Aug 11 05:49:32.015 UTC 
TenGigE0/9/0/0                 unassigned      Down            Down     default  
TenGigE0/9/0/1                 unassigned      Down            Down     default  
TenGigE0/9/0/2                 unassigned      Down            Down     default  
TenGigE0/9/0/3                 unassigned      Down            Down     default  
TwentyFiveGigE0/9/0/4          120.0.9.2       Up              Up       vpn10    
TwentyFiveGigE0/9/0/5          120.0.9.1       Up              Up       vpn9     
TwentyFiveGigE0/9/0/6          120.0.1.2       Down            Down     vpn2     
TwentyFiveGigE0/9/0/7          120.0.0.1       Up              Up       vpn1   
RP/0/RP1/CPU0:N560-7#</code>
</pre>
</div>

Now, we configure slot 7 QUAD-1 in 10G mode and QUAD-2 in 50G mode. Logically, we will see 4 ports TenGigE in 0-3 and two ports FiftyGigE in 4 and 6:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:N560-7#show run | inc "hw-module|mode"                  
Wed Aug 11 05:52:32.023 UTC
Building configuration...
hw-module quad 1 slot 7
 mode 10g
hw-module quad 2 slot 7
 mode 50
hw-module quad 1 slot 9 
 mode 10g
RP/0/RP1/CPU0:N560-7#
RP/0/RP1/CPU0:N560-7#show ipv4 int brief | inc 0/7/0/                 
Wed Aug 11 05:54:12.009 UTC
TenGigE0/7/0/0                 unassigned      Down            Down     default 
TenGigE0/7/0/1                 120.0.7.2       Up              Up       vpn8    
TenGigE0/7/0/2                 120.0.3.2       Up              Up       vpn4    
TenGigE0/7/0/3                 120.0.4.1       Up              Up       vpn4    
FiftyGigE0/7/0/4               unassigned      down            down     default   
FiftyGigE0/7/0/6               unassigned      down            down     default   
RP/0/RP1/CPU0:N560-7#
RP/0/RP1/CPU0:N560-7#show inventory location 0/7
NAME: "0/7", DESCR: "Cisco NCS 560 8-port 25G Interface Module, SFP+/SFP28 optics" 
PID: N560-IMA-8Q/4L    , VID: V00, SN: xxxxxxxxxxx 
  
NAME: "TenGigE0/7/0/1", DESCR: "Cisco SFP+ 10G SR Pluggable Optics Module" 
PID: SFP-10G-SR-S      , VID: V01, SN: xxxxxxxxxxx 
  
NAME: "TenGigE0/7/0/2", DESCR: "Cisco SFP+ 10G CWDM 1610nm Pluggable Optics Module" 
PID: CWDM-SFP10G-1610  , VID: V01, SN: xxxxxxxxxxx 
  
NAME: "TenGigE0/7/0/3", DESCR: "Cisco SFP+ 10G BXU-I Pluggable Optics Module" 
PID: SFP-10G-BXU-I     , VID: V01, SN: xxxxxxxxxxx 
  
NAME: "FiftyGigE0/7/0/4", DESCR: "Unknown Pluggable Optics Module" 
PID: N/A               , VID: N/A, SN: xxxxxxx 
  
NAME: "FiftyGigE0/7/0/6", DESCR: "Unknown Pluggable Optics Module" 
PID: N/A               , VID: N/A, SN: xxxxxxx 
RP/0/RP1/CPU0:N560-7#
</code>
</pre>
</div>

Also introduced in IOS XR 7.4.1, the support of "Partial" 4x10G to support the insertion of N560-IMA-8Z(-L) in 40Gbps slots.  
Before 7.4.1, these IMA were only supported in the 80+Gbps slots: 4, 5, 7, 9, 10 and 11:

![560-7.png]({{site.baseurl}}/images/560-7.png){: .align-center}

Now, it's possible to insert the IMA in 40Gbps slots (2, 3, 12 and 13) and support half of the ports (and which ports will be disabled is dependent on the type of IMA):

![IMA-8Z.png]({{site.baseurl}}/images/IMA-8Z.png){: .align-center}

![IMA-8Z-L.png]({{site.baseurl}}/images/IMA-8Z-L.png){: .align-center}

In this example below, we inserted an IMA-8Z in slot 12 and an IMA-8Z-L in slot 13 (both 40Gbps). You can see the first ports are enabled for the 8Z and the last ports are enabled for the 8Z-L:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:N560-7#show platform 
Wed Aug 11 05:39:29.011 UTC
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/1/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/2/CPU0          A900-IMA8CS1Z-M            OPERATIONAL       NSHUT
0/4/CPU0          A900-IMA8Z                 OPERATIONAL       NSHUT
0/5/CPU0          A900-IMA8Z                 OPERATIONAL       NSHUT
0/7/CPU0          N560-IMA-8Q/4L             OPERATIONAL       NSHUT
0/9/CPU0          N560-IMA-8Q/4L             OPERATIONAL       NSHUT
0/10/CPU0         A900-IMA8Z                 OPERATIONAL       NSHUT
0/11/CPU0         A900-IMA8Z-L               OPERATIONAL       NSHUT
0/12/CPU0         <mark>A900-IMA8Z</mark>                 OPERATIONAL       NSHUT
0/13/CPU0         <mark>A900-IMA8Z-L</mark>               OPERATIONAL       NSHUT
0/RP0/CPU0        N560-RSP4-E(Standby)       IOS XR RUN        NSHUT
0/RP1/CPU0        N560-RSP4-E(Active)        IOS XR RUN        NSHUT
0/FT0/CPU0        N560-FAN-H                 OPERATIONAL       NSHUT
0/PM2/CPU0        A900-PWR1200-A             OPERATIONAL       NSHUT
RP/0/RP1/CPU0:N560-7#
RP/0/RP0/CPU0:ios#show ipv4 int brief | inc 0/12/0
Wed Aug 11 05:42:26.011 UTC
Interface                      IP-Address      Status          Protocol Vrf-Name
TenGigE0/12/0/0                unassigned      Shutdown        Down     default 
TenGigE0/12/0/1                unassigned      Down            Down     default 
TenGigE0/12/0/2                unassigned      Down            Down     default 
TenGigE0/12/0/3                unassigned      Down            Down     default 
RP/0/RP0/CPU0:ios#show ipv4 int brief | inc 0/13/0
Wed Aug 11 05:43:12.011 UTC 
Interface                      IP-Address      Status          Protocol Vrf-Name
TenGigE0/13/0/4                unassigned      Shutdown        Down     default 
TenGigE0/13/0/5                unassigned      Shutdown        Down     default 
TenGigE0/13/0/6                unassigned      Shutdown        Down     default 
TenGigE0/13/0/7                unassigned      Shutdown        Down     default 
RP/0/RP1/CPU0:N560-7#</code>
</pre>
</div>
