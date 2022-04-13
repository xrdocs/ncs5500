---
published: true
date: '2019-09-02 10:29 +0200'
title: 'BGP FlowSpec on NCS5500: A few tests on scale, rate and memory usage'
position: top
author: Nicolas Fevrier
excerpt: BGP Flowspec implementation and resource management on NCS5500 Series
tags:
  - iosxr
  - ncs5500
  - flowspec
  - bgpfs
---
{% include toc icon="table" title="BGP FlowSpec on NCS5500: Few Tests" %} 

**Update 1**: Correction on the hw-module profile ipv6-flowspec section 
{: .notice--info}

**Update 2**: Netscout simplified the ntp auto-mitigation, we ran the test with this new rule. Also, error on the Netbios ports has been fixed.
{: .notice--info}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Yosef published a couple of articles related to BGP FlowSpec implementation on the NCS5500 routers here:  
- [SupportForum: BGP Flowspec implementation on NCS5500 platforms](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443): [https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)
- [SupportForum: NCS5500 BGP flowspec packet matching criteria](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443): [https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)

Today, we will gather several questions from customers and we will use this opportunity to dig a bit deeper in the subtleties of this implementation:  
- presenting the memory spaces used to store the rules information and the statistics
- and running a couple of tests to identify the limits.

As a starter, I suggest three videos on Youtube that could answer most of your questions on the topic. The first two are relatively short, the last one will require a couple of hours of your time.

All the principles and details of the configuration:  
[Cisco NCS5500 Flowspec (Principles and Configuration) Part1](https://www.youtube.com/watch?v=dTgh0p9Vyns)

<iframe width="560" height="315" src="https://www.youtube.com/embed/dTgh0p9Vyns?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}
  

A simple demo of interoperability between Netscout / Arbor SP and NCS5500 to auto-mitigate an MemCacheD amplification attack:  
[Cisco NCS5500 Flowspec (Auto-Mitigation of a Memcached Attack) Part2](https://www.youtube.com/watch?v=iRPob7Ws2v8)

<iframe width="560" height="315" src="https://www.youtube.com/embed/iRPob7Ws2v8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

Finally, the CiscoLive session dedicated to BGP FlowSpec. A deepdive in the technology:  
[BRKSPG 3012 - Leveraging BGP Flowspec to protect your infrastructure](https://www.youtube.com/watch?v=dbsNf8DcNRQ)

<iframe width="560" height="315" src="https://www.youtube.com/embed/dbsNf8DcNRQ" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

## Specific NCS5500 implementation

First reminder: the support is limited today (September 2019) to the platforms based on Jericho+ NPU and External TCAM (OP: Optimus Prime).

BGP FlowSpec being implemented in ingress, the distinction between line card is important only where the packets are received. What is used to egress the traffic is not relevant.  
We support BGP FS on the following products:  
- NCS55A1-36H-SE-S
- NCS55A2-MOD-SE-S (the one we are using for these tests)
- NC55-36X100G-A-SE line card
- NC55-MOD-A-SE-S line card

For the most part, the implementation is identical to what has been done on the ASR9000, CRS and NCS6000 platforms.  
You can refer to the configuration guide on the ASR9000 and use the examples available from multiple sources. 

In the next parts, you'll find aspect that are specific to the NCS5500:

### Recirculation

When packets are matched by a BGP FS rule, they will be recirculated. It's required to permit the accounting of the matched packets.  

### IPv6 specific mode

BGP FS for IPv6 requires a specific hardware profile.

It will impact the overall performance. That means all packets, handled or not by the BGP FlowSpec rules, will be treated at a maximum of 700MPPS instead of the nominal 835MPPS.

You need to enable the following profile as described below:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE(config)#hw-module profile flowspec ?
  v6-enable  Configure support for v6 flowspec
RP/0/RP0/CPU0:Peyto-SE(config)#hw-module profile flowspec v6-enable ?
  location  Location of flowspec config
RP/0/RP0/CPU0:Peyto-SE(config)#hw-module profile flowspec v6-enable
RP/0/RP0/CPU0:Peyto-SE(config)#commit
RP/0/RP0/CPU0:Peyto-SE(config)#</code>
</pre>
</div>

To be enabled, the profile needs a reload of the line cards or the entire system.

### Interface support

Yosef covered it in the supportforum blog but it's important to remind that BGP flowspec is activate on L3 interface but will NOT process packets when received from GRE tunnel, or on BVI interface. Also, BGP flowspec is NOT supported with multicast traffic.

## Test setup

__Config Route Generator / Controller__

<div class="highlighter-rouge">
<pre class="highlight">
<code>router bgp 100
bgp_id 192.168.100.151
neighbor 192.168.100.217 remote-as 100
neighbor 192.168.100.217 update-source 192.168.100.151
capability ipv4 flowspec

network 1 ipv4 flowspec
network 1 dest 2.2.2.0/24 source 3.3.0.0/16 protocol 6 dest-port 8080
network 1 count 4000 dest-incr
ext_community 1 traffic-rate:1:0</code>
</pre>
</div>

__Config Router / Client__ :

We are using an NCS55A2MOD router with External TCAM:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh plat

Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/1             NC55-MPA-4H-S              OK
0/0/2             NC55-MPA-12T-S             OK
0/RP0/CPU0        NCS-55A2-MOD-SE-S(Active)  IOS XR RUN        NSHUT
0/RP0/NPU0        Slice                      UP
0/FT0             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT1             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT2             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT3             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT4             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT5             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT6             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/FT7             NC55-A2-FAN-FW             OPERATIONAL       NSHUT
0/PM0             NC55-1200W-ACFW            OPERATIONAL       NSHUT
0/PM1             NC55-1200W-ACFW            FAILED            NSHUT
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

And the configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
router bgp 100
 address-family ipv4 flowspec
 !
 neighbor 192.168.100.151
  remote-as 100
  update-source MgmtEth0/RP0/CPU0/0
  !
  address-family ipv4 flowspec
   route-policy PERMIT-ANY in
   route-policy PERMIT-ANY out
  !
 !
flowspec
 local-install interface-all
!</code>
</pre>
</div>

## Scale Tests

### 3000 rules 

From the controller, we advertise 3000 simple rules (which is the level of support on the IOS XR routers) and we will use this opportunity to check the resources consumed. The following commands can be used for normal operation and troubleshooting.

We verify the advertisement at the BGP peer level first:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh bgp ipv4 flowspec sum

BGP router identifier 1.1.1.111, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 97804
BGP main routing table version 97804
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker           97804      97804      97804      97804       97804           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100     802     463    97804    0    0 00:00:11       <mark>3000</mark>

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

We also verify that the rules are properly received:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#show policy-map transient type pbr pmap-name __bgpfs_default_IPv4
policy-map type pbr __bgpfs_default_IPv4
 handle:0x36000002
 table description: L3 IPv4 and IPv6
 class handle:0x76004f03  sequence 1024
   match destination-address ipv4 2.2.2.0 255.255.255.0
   match source-address ipv4 3.3.0.0 255.255.0.0
   match protocol tcp
   match destination-port 8080
  drop
 !
 class handle:0x76004f04  sequence 2048
   match destination-address ipv4 2.2.3.0 255.255.255.0
   match source-address ipv4 3.3.0.0 255.255.0.0
   match protocol tcp
   match destination-port 8080
  drop
 !
 ...
</code>
</pre>
</div>

On the flowspec side too:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4 detail

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=8080
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    Statistics                        (packets/bytes)
      Matched             :                   0/0
      Transmitted         :                   0/0
      Dropped             :                   0/0
  Flow           :Dest:2.2.3.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=8080
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    Statistics                        (packets/bytes)
      Matched             :                   0/0
      Transmitted         :                   0/0
      Dropped             :                   0/0
  Flow           :Dest:2.2.4.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=8080
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    Statistics                        (packets/bytes)
      Matched             :                   0/0
      Transmitted         :                   0/0
      Dropped             :                   0/0
  Flow           :Dest:2.2.5.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=8080
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    Statistics                        (packets/bytes)
      Matched             :                   0/0
      Transmitted         :                   0/0
      Dropped             :                   0/0
...
</code>
</pre>
</div>

To be passed from IOS XR to the hardware, we are using the DPA/OFA table "ippbr":

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh dpa resources ippbr loc 0/0/cPU0

"ippbr" OFA Table (Id: 137, Scope: Global)
--------------------------------------------------
                          NPU ID: NPU-0
                          In Use: <mark>3000</mark>
                 Create Requests
                           Total: 3000
                         Success: 3000
                 Delete Requests
                           Total: 1000
                         Success: 1000
                 Update Requests
                           Total: 0
                         Success: 0
                    EOD Requests
                           Total: 0
                         Success: 0
                          Errors
                     HW Failures: 0
                Resolve Failures: 0
                 No memory in DB: 0
                 Not found in DB: 0
                    Exists in DB: 0
      Reserve Resources Failures: 0
      Release Resources Failures: 0
       Update Resources Failures: 0

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

The BGP FlowSpec rules are stored in external TCAM in a specific zone, different from the one used for IPv4 and IPv6 prefixes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         1096       <mark>3000</mark>    126  <mark>INGRESS_FLOWSPEC_IPV4</mark>
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

Nothing will be used in the other most common resources: LPM, LEM, IPv4/IPv6 eTCAM or iTCAM. You can verify it with "sh contr npu resources all loc 0/0/CPU0"

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources all loc 0/0/CPU0

HW Resource Information
    Name                            : <mark>lem</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 0        (0 %)
        iproute                     : 0        (0 %)
        ip6route                    : 0        (0 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

HW Resource Information
    Name                            : <mark>lpm</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 329283
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 4        (0 %)
        iproute                     : 0        (0 %)
        ip6route                    : 0        (0 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

HW Resource Information
    Name                            : <mark>encap</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 104000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 0        (0 %)
        ipnh                        : 0        (0 %)
        ip6nh                       : 0        (0 %)
        mplsnh                      : 0        (0 %)

HW Resource Information
    Name                            : <mark>ext_tcam_ipv4</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 6        (0 %)
        iproute                     : 9        (0 %)

HW Resource Information
    Name                            : <mark>fec</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 126976
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 15       (0 %)
        ipnhgroup                   : 7        (0 %)
        ip6nhgroup                  : 2        (0 %)
        edpl                        : 0        (0 %)
        limd                        : 0        (0 %)
        punt                        : 4        (0 %)
        iptunneldecap               : 0        (0 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ipnh                        : 0        (0 %)
        ip6nh                       : 0        (0 %)
        mplsmdtbud                  : 0        (0 %)
        ipvrf                       : 1        (0 %)
        ippbr                       : 0        (0 %)
        redirectvrf                 : 0        (0 %)
        erp                         : 0        (0 %)

HW Resource Information
    Name                            : <mark>ecmp_fec</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 4096
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 0        (0 %)
        ipnhgroup                   : 0        (0 %)
        ip6nhgroup                  : 0        (0 %)

HW Resource Information
    Name                            : <mark>ext_tcam_ipv6</mark>

OOR Information
    NPU-0
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 3        (0 %)
        ip6route                    : 9        (0 %)

RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu internaltcam location 0/0/CPU0

Internal TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      160b   flp-tcam    2045     0       0
0    1      160b   pmf-0       1959     58      7    INGRESS_LPTS_IPV4
0    1      160b   pmf-0       1959     8       14   INGRESS_RX_ISIS
0    1      160b   pmf-0       1959     16      27   INGRESS_QOS_IPV4
0    1      160b   pmf-0       1959     6       29   INGRESS_QOS_MPLS
0    1      160b   pmf-0       1959     1       36   INGRESS_EVPN_AA_ESI_TO_FBN_DB
0    2      160b   pmf-0       1975     40      17   INGRESS_ACL_L3_IPV4
0    2      160b   pmf-0       1975     33      30   INGRESS_QOS_L2
0    3      160b   egress_acl  2030     18      3    EGRESS_QOS_MAP
0    4\5    320b   pmf-0       1984     49      8    INGRESS_LPTS_IPV6
0    4\5    320b   pmf-0       1984     15      28   INGRESS_QOS_IPV6
0    6      160b   Free        2048     0       0
0    7      160b   Free        2048     0       0
0    8      160b   Free        2048     0       0
0    9      160b   Free        2048     0       0
0    10     160b   Free        2048     0       0
0    11     160b   Free        2048     0       0
0    12     160b   flp-tcam    125      0       0
0    13     160b   pmf-1       9        54      13   INGRESS_RX_L2
0    13     160b   pmf-1       9        13      23   INGRESS_MPLS
0    13     160b   pmf-1       9        46      74   INGRESS_BFD_IPV4_NO_DESC_TCAM_T
0    13     160b   pmf-1       9        4       86   SRV6_END
0    13     160b   pmf-1       9        2       95   INGRESS_IP_DISABLE
0    14     160b   egress_acl  120      8       6    EGRESS_L3_QOS_MAP
0    15     160b   Free        128      0       0
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

The BGP Flowspec rules will consume statistic entries.

Before the advertisement of the rules:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance 0 loc 0/0/CPU0

System information for NPU 0:
  Counter processor configuration profile: Default
  Next available counter processor:        6

Counter processor: 0                        | Counter processor: 1
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Trap                       113     300  |     Trap                       110     300
    Policer (QoS)               32    6976  |     Policer (QoS)                0    6976
    ACL RX, LPTS               <mark>202</mark>     915  |     ACL RX, LPTS               <mark>202</mark>     915
                                            |
                                            |
Counter processor: 2                        | Counter processor: 3
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    VOQ                         67    8191  |     VOQ                         67    8191
                                            |
                                            |
Counter processor: 4                        | Counter processor: 5
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 6                        | Counter processor: 7
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 8                        | Counter processor: 9
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 10                       | Counter processor: 11
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    L3 RX                        0    1638  |     L3 RX                        0    1638
    L2 RX                        0    8192  |     L2 RX                        0    8192
                                            |
                                            |
Counter processor: 12                       | Counter processor: 13
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16383  |     Interface TX                 0   16383
                                            |
                                            |
Counter processor: 14                       | Counter processor: 15
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16384  |     Interface TX                 0   16384
                                            |
                                            |
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

We highlighted the "ACL RX, LPTS" which will contain the counters for Flowspec.  
Before injecting the rules, we are already consuming 202 entries. It will be our reference point.

And now after the learning of 3000 rules:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance 0 location all

HW Stats Information For Location: 0/0/CPU0

System information for NPU 0:
  Counter processor configuration profile: Default
  Next available counter processor:        6

Counter processor: 0                        | Counter processor: 1
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Trap                       113     300  |     Trap                       110     300
    Policer (QoS)               32    6976  |     Policer (QoS)                0    6976
    ACL RX, LPTS               <mark>914</mark>     915  |     ACL RX, LPTS               <mark>914</mark>     915
                                            |
                                            |
Counter processor: 2                        | Counter processor: 3
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    VOQ                         67    8191  |     VOQ                         67    8191
                                            |
                                            |
Counter processor: 4                        | Counter processor: 5
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    ACL RX, LPTS              <mark>2288</mark>    8192  |     ACL RX, LPTS              <mark>2288</mark>    8192
                                            |
                                            |
Counter processor: 6                        | Counter processor: 7
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 8                        | Counter processor: 9
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 10                       | Counter processor: 11
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    L3 RX                        0    1638  |     L3 RX                        0    1638
    L2 RX                        0    8192  |     L2 RX                        0    8192
                                            |
                                            |
Counter processor: 12                       | Counter processor: 13
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16383  |     Interface TX                 0   16383
                                            |
                                            |
Counter processor: 14                       | Counter processor: 15
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16384  |     Interface TX                 0   16384
                                            |
                                            |
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

In Counter Processor 0: we used to consume 202 entries before the BGP FS rules and we have now 914, so, 712 entries have allocated to Flowspec.  

In Counter Processor 4: we allocated 2288 new entries.  

So, in total, we have 2288 + 712 = 3000 entries which is in-line with the expectation.

**Note**: This number 3000 is the validated scale on all the IOS XR platforms. It does not mean that some systems couldn't go higher. It will depend on the platforms and the software releases. But 3000 simple rules are guaranteed. The rest of the tests performed below will try to answer specific questions from customers (during CPOC or for production), but it's only for information. Results may vary depending on platform and software release.
{: .notice--info}

So, what happens if we inject 4000, 6000 or 9000 rules?  

### 4000 rules

Let's see what will happen if we push further. We start with 4000 rules of the same kind than used in the former test.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         96       <mark>4000</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh dpa resources ippbr loc 0/0/CPU0

"ippbr" OFA Table (Id: 137, Scope: Global)
--------------------------------------------------
                          NPU ID: NPU-0
                          In Use: <mark>4000</mark>
                 Create Requests
                           Total: 7000
                         Success: 7000
                 Delete Requests
                           Total: 4118
                         Success: 4118
                 Update Requests
                           Total: 0
                         Success: 0
                    EOD Requests
                           Total: 0
                         Success: 0
                          Errors
                     <mark>HW Failures: 0</mark>
                Resolve Failures: 0
                 No memory in DB: 0
                 Not found in DB: 0
                    Exists in DB: 0
      Reserve Resources Failures: 0
      Release Resources Failures: 0
       Update Resources Failures: 0

RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance 0 location all

HW Stats Information For Location: 0/0/CPU0

System information for NPU 0:
  Counter processor configuration profile: Default
  Next available counter processor:        6

Counter processor: 0                        | Counter processor: 1
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Trap                       113     300  |     Trap                       110     300
    Policer (QoS)               32    6976  |     Policer (QoS)                0    6976
    ACL RX, LPTS               <mark>912</mark>     915  |     ACL RX, LPTS               <mark>912</mark>     915
                                            |
                                            |
Counter processor: 2                        | Counter processor: 3
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    VOQ                         67    8191  |     VOQ                         67    8191
                                            |
                                            |
Counter processor: 4                        | Counter processor: 5
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    ACL RX, LPTS              <mark>3287</mark>    8192  |     ACL RX, LPTS              <mark>3287</mark>    8192
                                            |
                                            |
Counter processor: 6                        | Counter processor: 7
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 8                        | Counter processor: 9
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 10                       | Counter processor: 11
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    L3 RX                        0    1638  |     L3 RX                        0    1638
    L2 RX                        0    8192  |     L2 RX                        0    8192
                                            |
                                            |
Counter processor: 12                       | Counter processor: 13
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16383  |     Interface TX                 0   16383
                                            |
                                            |
Counter processor: 14                       | Counter processor: 15
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16384  |     Interface TX                 0   16384
                                            |
                                            |
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

It looks like 4000 entries were received quickly and didn't trigger any error.

### 6000 rules

Moving the cursor to 6000 rules now, twice the supported level.

The BGP part is learnt almost instantly.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh bgp ipv4 flowspec  sum

BGP router identifier 1.1.1.111, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 132804
BGP main routing table version 132804
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          132804     126804     132804     132804      126804           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100     989     523   126804    0    0 00:00:33       <mark>6000</mark>

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

On the hardware side, the first 4200 rules are programmed in a few seconds then it progresses much more slowly:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         4940     4276    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

It will take several minutes to program the remaining 2000ish rules.

Eventually, rules will be programmed and the DPA part doesn't show any error despite the very long time it takes.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         4240     <mark>6000</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance 0 location all

HW Stats Information For Location: 0/0/CPU0

System information for NPU 0:
  Counter processor configuration profile: Default
  Next available counter processor:        4

Counter processor: 0                        | Counter processor: 1
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Trap                       113     300  |     Trap                       110     300
    Policer (QoS)               32    6976  |     Policer (QoS)                0    6976
    ACL RX, LPTS               <mark>915</mark>     915  |     ACL RX, LPTS               <mark>915</mark>     915
                                            |
                                            |
Counter processor: 2                        | Counter processor: 3
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    VOQ                         67    8191  |     VOQ                         67    8191
                                            |
                                            |
Counter processor: 4                        | Counter processor: 5
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 6                        | Counter processor: 7
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    ACL RX, LPTS              <mark>5287</mark>    8192  |     ACL RX, LPTS              <mark>5287</mark>    8192
                                            |
                                            |
Counter processor: 8                        | Counter processor: 9
  State: Free                               |   State: Free
                                            |
                                            |
Counter processor: 10                       | Counter processor: 11
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    L3 RX                        0    1638  |     L3 RX                        0    1638
    L2 RX                        0    8192  |     L2 RX                        0    8192
                                            |
                                            |
Counter processor: 12                       | Counter processor: 13
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16383  |     Interface TX                 0   16383
                                            |
                                            |
Counter processor: 14                       | Counter processor: 15
  State: In use                             |   State: In use
                                            |
  Application:              In use   Total  |   Application:              In use   Total
    Interface TX                 0   16384  |     Interface TX                 0   16384
                                            |
                                            |
RP/0/RP0/CPU0:Peyto-SE#sh dpa resources ippbr loc 0/0/CPU0

"ippbr" OFA Table (Id: 137, Scope: Global)
--------------------------------------------------
                          NPU ID: NPU-0
                          In Use: <mark>6000</mark>
                 Create Requests
                           Total: 179286
                         Success: 179286
                 Delete Requests
                           Total: 173286
                         Success: 173286
                 Update Requests
                           Total: 0
                         Success: 0
                    EOD Requests
                           Total: 0
                         Success: 0
                          Errors
                     HW Failures: <mark>0</mark>
                Resolve Failures: 0
                 No memory in DB: 0
                 Not found in DB: 0
                    Exists in DB: 0
      Reserve Resources Failures: 0
      Release Resources Failures: 0
       Update Resources Failures: 0

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

### 9000 rules

Ok, one last try... This time with 9000 rules. Three times the officially supported scale.

Like we noticed for the former test with 6000 rules, the BGP part is going pretty fast, the programming goes to 4200 rules quickly and then learns the routes slowly.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh bgp ipv4 flowspec sum

BGP router identifier 1.1.1.111, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 163804
BGP main routing table version 163804
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          163804     154804     163804     163804      154804           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100    1174     593   154804    0    0 00:02:45       9000

RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         4997     4219    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

This time, we pushed too far and exceeded the memory allocations.  
The DPA/OFA is showing error messages which proves it was not able to program the entry in hardware.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         4406     8906    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh dpa resources ippbr loc 0/0/CPU0

"ippbr" OFA Table (Id: 137, Scope: Global)
--------------------------------------------------
                          NPU ID: NPU-0
                          In Use: 8906
                 Create Requests
                           Total: 867909
                         Success: 867374
                 Delete Requests
                           Total: 858909
                         Success: 858468
                 Update Requests
                           Total: 0
                         Success: 0
                    EOD Requests
                           Total: 0
                         Success: 0
                          Errors
                     HW Failures: 535
                Resolve Failures: 0
                 No memory in DB: 0
                 Not found in DB: 441
                    Exists in DB: 0
      Reserve Resources Failures: 0
      Release Resources Failures: 0
       Update Resources Failures: 0

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

We are seeing the router is not behaving erratically (crash or memory dumps), it just refuses to program more entries in the memory and increments the DPA Hw errors counters.

I have to re-iterate: the officially tested, it means, supported scale for BGP Flowspec is 3000 rules. 

We were able to push to 4000 with this platform with no noticeable problem, to 6000 with a very low programming rate in the last part but not to 9000. But it doesn't prove anything, just that it doesn't badly impair the router.

The results may be different on a different NCS5500 platform or a different IOS XR version. So, please take all this with a grain of salt.

## Latest Scale

With **IOS-XR 7.6.1**, we can now assign **32K BGP Flowspec entries**, thus increasing the number of matches and actions covered. In earlier releases, you could configure 16K BGP Flowspec entries. BGP Flowspec entries up to 32K are supported only on Cisco NCS 5700 series fixed port routers and the Cisco NCS 5500 series routers that have the Cisco NC57 line cards that are installed and operating in the native mode. BGP Flowspec can scale up to 32K entries only when you enable the l3max-se profile. 

More details on the scale can be found [here.](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/bgp/76x/b-bgp-cg-ncs5500-76x/implementing-bgp.html#Cisco_Concept.dita_38717a1a-6b0b-4fa1-96cb-9383ac932f80)


## Session limit configuration

_Is it possible to limit the number of rules received per session or globally?_ 

We can configure the "maximum-prefix" under the neighbor statement to limit the number of advertised (received) rules for a given session. But it's not possible to globally limit the number of rules to a specific value.

The only workaround will consist in using a single BGP FS session from the client to a route-reflector.  
{: .notice--info}

The max-prefix feature is directly inherited from the BGP world and benefits to Flowspec without specific adaptation.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE(config)#router bgp 100
RP/0/RP0/CPU0:Peyto-SE(config-bgp)# neighbor 192.168.100.151
RP/0/RP0/CPU0:Peyto-SE(config-bgp-nbr)#  address-family ipv4 flowspec
RP/0/RP0/CPU0:Peyto-SE(config-bgp-nbr-af)#maximum-prefix 1010 75
RP/0/RP0/CPU0:Peyto-SE(config-bgp-nbr-af)#commit</code>
</pre>
</div>

We advertise 1000 rules, it only generates a warning message:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Jul 15 00:56:58.887 UTC: bgp[1084]: %ROUTING-BGP-5-ADJCHANGE : 
neighbor 192.168.100.151 Up (VRF: default) (AS: 100)
RP/0/RP0/CPU0:Jul 15 00:56:58.888 UTC: bgp[1084]: %ROUTING-BGP-5-NSR_STATE_CHANGE : 
Changed state to Not NSR-Ready
RP/0/RP0/CPU0:Jul 15 00:56:59.147 UTC: bgp[1084]: %ROUTING-BGP-5-MAXPFX : 
No. of IPv4 Flowspec prefixes received from 192.168.100.151 has reached 758, max 1010</code>
</pre>
</div>

If we push to 1020 rules:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Jul 15 00:59:55.549 UTC: bgp[1084]: %ROUTING-BGP-4-MAXPFXEXCEED : 
No. of IPv4 Flowspec prefixes received from 192.168.100.151: <mark>1011 exceed limit 1010</mark>
RP/0/RP0/CPU0:Jul 15 00:59:55.549 UTC: bgp[1084]: %ROUTING-BGP-5-ADJCHANGE : 
neighbor 192.168.100.151 Down - <mark>Peer exceeding maximum prefix limit (CEASE notification sent - maximum number of prefixes reached)</mark> (VRF: default) (AS: 100)

RP/0/RP0/CPU0:Peyto-SE#sh bgp ipv4 flowspec sum

BGP router identifier 1.1.1.111, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 176824
BGP main routing table version 176824
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          176824     176824     176824     176824      176824           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100    1243     649        0    0    0 00:00:33 <mark>Idle (PfxCt)</mark>

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

Note that by default, it will be necessary to clear the bgp session to "unstuck" it from idle state.  
Also, other options exist to restart it automatically after a few minutes, to ignore the extra rules or to simply generate a warning message:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE(config-bgp-nbr-af)#maximum-prefix 1010 75 ?
  discard-extra-paths  Discard extra paths when limit is exceeded
  restart              Restart time interval
  warning-only         Only give warning message when limit is exceeded
RP/0/RP0/CPU0:Peyto-SE(config-bgp-nbr-af)#</code>
</pre>
</div>

## Verification of the resource used with complex rules

In the tests above, we  used a simple rule made of:
- source prefix
- destination prefix
- protocol UDP
- port 8080

From the generator, it's represented as:

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 dest 2.2.2.0/24 source 3.3.0.0/16 protocol 6 dest-port 8080</code>
</pre>
</div>

Which is received on the client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=8080
    Actions      :Traffic-rate: 0 bps  (bgp.1)</code>
</pre>
</div>

This simple rule will use a single entry in our external TCAM bank 11.

Now, let's try to identify how much space other rules will consume.

### ICMP type / code

On the controller, we advertise 100 rules with source, destination, ICMP type and code, and an increase of the destination.

On the Controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 2.2.2.0/24 source 3.3.0.0/16
network 1 icmp-type 3 icmp-code 16
network 1 count 100 dest-incr</code>
</pre>
</div>

On the Client/Router:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,ICMPType:=3,ICMPCode:=16
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:2.2.3.0/24,Source:3.3.0.0/16,ICMPType:=3,ICMPCode:=16
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | FLOWSPEC
0    11     320b   FLP         3996     <mark>100</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>301</mark>     915  |     ACL RX, LPTS               <mark>301</mark>     915
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

100 rules occupy 100 entries in the eTCAM and in the stats DB.  
So one for one.

### Packet size

We define on the controller a set of 100 rules with address source and destination, protocol TCP, destination port 123 and larger than 400 bytes:

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 2.2.2.0/24 source 3.3.0.0/16 protocol 6 dest-port 123
network 1 packet-len >=400
network 1 count 100 dest-incr</code>
</pre>
</div>

On the client side:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=123,Length:>=400
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:2.2.3.0/24,Source:3.3.0.0/16,Proto:=6,DPort:=123,Length:>=400
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOWSPEC
0    11     320b   FLP         3096     <mark>1000</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>300</mark>     915  |     ACL RX, LPTS               <mark>300</mark>     915
RP/0/RP0/CPU0:Peyto-SE#
</code>
</pre>
</div>

On the statistic side, one rule occupies one entry. But on the eTCAM, each rule will consume 10 entries.

Let's try to see if different packet sizes will show different memory occupation.

<div class="highlighter-rouge">
<pre class="highlight">
<code>**network 1 packet-len >=255**

RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         3196     <mark>900</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#

**network 1 packet-len >=256**

RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         3296     <mark>800</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#

**network 1 packet-len >=257**

RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         2596     <mark>1500</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#

**network 1 packet-len >=512**

RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         3396     <mark>700</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#

</code>
</pre>
</div>

Clearly, (packet) size matters:

| <= X Bytes | eTCAM Entries for one rule | <= X Bytes | eTCAM Entries for one rule |
|:------:|:------:|:------:|:------:|
| 120 | 10 | 245 | 11 |
| 121 | 12 | 246 | 10 |
| 122 | 11 | 247 | 10 |
| 123 | 11 | 248 | 9 |
| 124 | 10 | 249 | 11 |
| 125 | 11 | 250 | 10 |
| 126 | 10 | 251 | 10 |
| 127 | 10 | 252 | 9 |
| 128 | 9 | 253 | 10 |
| 129 | 15 | 254 | 9 |
| 130 | 14 | 255 | 9 |
| 131 | 14 | 256 | 8 |
| 132 | 13 | 257 | 15 |
| 133 | 14 | 258 | 14 |
| 134 | 13 | 259 | 14 |

Based on these couples of examples, to optimize the memory utilization, it's advised to use power of twos.

### Fragmented

In this example, we only use source and destination, and the indication the packets are fragmented.

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 2.2.2.0/24 source 3.3.0.0/16 protocol 17 fragment (isf)
network 1 count 100 dest-incr</code>
</pre>
</div>

On the router:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,Proto:=17,Frag:~IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:2.2.3.0/24,Source:3.3.0.0/16,Proto:=17,Frag:~IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         3896     <mark>200</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#s
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>300</mark>     915  |     ACL RX, LPTS               <mark>300</mark>     915
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

So, a simple rule with source and destination address and fragment flag will use one stats entry and two eTCAM entries.

### TCP SYN

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 2.2.2.0/24 source 3.3.0.0/16 protocol 6 tcp-flags *(syn)
network 1 count 100 dest-incr</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:2.2.2.0/24,Source:3.3.0.0/16,Proto:=6,TCPFlags:=0x02
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:2.2.3.0/24,Source:3.3.0.0/16,Proto:=6,TCPFlags:=0x02
    Actions      :Traffic-rate: 0 bps  (bgp.1)

RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/cpu0 | i FLOW
0    11     320b   FLP         3996     <mark>100</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>302</mark>     915  |     ACL RX, LPTS               <mark>302</mark>     915
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

For TCP SYNs, one stats and one eTCAM entry per rule.

## Arbor auto-mitigation

When Netscout / Arbor SP is used as a Flowspec controller, it can generate auto-mitigation rules such as:  
chargen, cldap, mdns, memcached, mssql, ripv1, rpcbind, ssdp, netbios, snmp, dns, l2tp, ntp and frags.

![arbor.png]({{site.baseurl}}/images/arbor.png){: .align-center}

### First group: unique source-port

- chargen: dest 7.7.7.7/32 protocol 17 source-port 19
- cldap: dest 7.7.7.7/32 protocol 17 source-port 389
- mdns: dest 7.7.7.7/32 protocol 17 source-port 5353
- memcached: dest 7.7.7.7/32 protocol 17 source-port 11211
- mssql: dest 7.7.7.7/32 protocol 17 source-port 1434
- ripv1: dest 7.7.7.7/32 protocol 17 source-port 520
- rpcbind: dest 7.7.7.7/32 protocol 17 source-port 111
- ssdp: dest 7.7.7.7/32 protocol 17 source-port 1900

On the controller:

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 7.7.7.7/32 protocol 17 source-port 19
network 1 count 100 dest-incr</code>
</pre>
</div>

On the router/client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4
AFI: IPv4
  Flow           :<mark>Dest:7.7.7.7/32,Proto:=17,SPort:=19</mark>
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.8/32,Proto:=17,SPort:=19
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3996     <mark>100</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>303</mark>     915  |     ACL RX, LPTS               <mark>303</mark>     915
RP/0/RP0/CPU0:Peyto-SE#
</code>
</pre>
</div>

--> For all these cases, it will consume one stats entry and one eTCAM per rule.

### Second group: dual source-port

- netbios: dest 7.7.7.7/32 protocol 17 source-port {137 138}
- snmp: dest 7.7.7.7/32 protocol 17 source-port {161 162}

Controller config: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 7.7.7.7/32 protocol 17 source-port {137 138}
network 1 count 100 dest-incr</code>
</pre>
</div>

On the router/client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=137|=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.8/32,Proto:=17,SPort:=137|=138
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3896     <mark>200</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>303</mark>     915  |     ACL RX, LPTS               <mark>303</mark>     915
RP/0/RP0/CPU0:Peyto-SE#
</code>
</pre>
</div>

--> these cases are consuming one stats entry and two eTCAM entries per rule.

### Third group: packet length

- dns: dest 7.7.7.7/32 protocol 17 source-port 53 packet-len {>=768}

On the controller side: 

<div class="highlighter-rouge">
<pre class="highlight">
<code>network 1 ipv4 flowspec
network 1 dest 7.7.7.7/32 protocol 17 source-port 53 packet-len {>=768}
network 1 count 100 dest-incr</code>
</pre>
</div>

On the router/client:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=53,Length:>=768
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.8/32,Proto:=17,SPort:=53,Length:>=768
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3396     <mark>700</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>302</mark>     915  |     ACL RX, LPTS               <mark>302</mark>     915
RP/0/RP0/CPU0:Peyto-SE#
</code>
</pre>
</div>

--> with this range (larger than 768), it consumes one stats entry and 7 eTCAM entries per rule.

- l2tp: dest 7.7.7.7/32 protocol 17 source-port 1701 packet-len {>=500}

We check if the "larger than 500" makes a significant difference:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3196     <mark>900</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

--> yes, each rule will consume 9 eTCAM entries here. Some optimization is possible but it will not change fundamentally the scale.

- ntp: dest 7.7.7.7/32 protocol 17 source-port 123 packet-len {>=1 and<=35 >=37 and<=45 >=47 and<=75 >=77 and<=219 >=221 and<=65535}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         796      <mark>3300</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>302</mark>     915  |     ACL RX, LPTS               <mark>302</mark>     915
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

--> each rule here will consume one stats entry and 33 eTCAM entries.

**Update**: In latest version, NetScout modified the NTP auto-mitigation rule to use only the ranges 1-75,77-550
{: .notice--info}

- ntp: dest 7.7.7.7/32 protocol 17 source-port 123 packet-len {>=1 and<=75 >=77 and<=550}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3096     <mark>1000</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 |$
    ACL RX, LPTS               <mark>301</mark>     915  |     ACL RX, LPTS               <mark>301</mark>     915
RP/0/RP0/CPU0:Peyto-SE#
</code>
</pre>
</div>

With these two ranges, each rule will consume 10 entries in the eTCAM (and still one in the stats).

### Last group: frag

- udp-frag: dest 7.7.7.7/32 protocol 17 fragment (isf)

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Proto:=17,Frag:~IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
  Flow           :Dest:7.7.7.8/32,Proto:=17,Frag:~IsF
    Actions      :Traffic-rate: 0 bps  (bgp.1)
    ...
RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam loc 0/0/CPU0 | i FLOWSPEC
0    11     320b   FLP         3896     <mark>200</mark>     126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#sh contr npu resource stats instance all loc 0/0/CPU0 | i ACL
    ACL RX, LPTS               <mark>302</mark>     915  |     ACL RX, LPTS               <mark>302</mark>     915
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

### To summarize

| Auto-Mitigation | eTCAM Entries |
|:------:|:------:|
| chargen | 1 |
| cldap | 1 |
| mdns | 1 |
| memcached | 1 |
| mssql | 1 |
| ripv1 | 1 |
| rpcbind | 1 |
| ssdp | 1 |
| netbios | 2 |
| snmp | 2 |
| dns | 7 |
| l2tp | 9 |
| ntp | 33 |
| UDP frag | 2 |

## Programming rate

To measure the number of rules we can program per second, we are using a very rudimentary method based on show command timestamps.  

After establishing the flowspec session, I will type "sh contr npu externaltcam location 0/0/CPU0" regularly and collect the number of entries in the bank ID 11, I will also note down the timing of the session, and convert it in milliseconds.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu externaltcam location 0/0/CPU0
<mark>Sun Jul 14 23:35:44.252 UTC</mark>
External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         6481603  6       0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         2389864  3       3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      320b   FLP         4067     29      5    IPv6 MC
0    5      80b    FLP         4096     0       82   INGRESS_IPV4_SRC_IP_EXT
0    6      80b    FLP         4096     0       83   INGRESS_IPV4_DST_IP_EXT
0    7      160b   FLP         4096     0       84   INGRESS_IPV6_SRC_IP_EXT
0    8      160b   FLP         4096     0       85   INGRESS_IPV6_DST_IP_EXT
0    9      80b    FLP         4096     0       86   INGRESS_IP_SRC_PORT_EXT
0    10     80b    FLP         4096     0       87   INGRESS_IPV6_SRC_PORT_EXT
0    11     320b   FLP         4351     <mark>4865</mark>    126  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

I can extract the following chart and diagram:

| Timing (ms) | eTCAM Entries |
|:------:|:------:|
| 38610 | 549 |
| 39551 | 774 |
| 40320 | 950 |
| 41128 | 1150 |
| 41979 | 1352 |
| 42680 | 1532 |
| 43384 | 1700 |
| 44039 | 1850 |
| 44673 | 2003 |
| 45312 | 2159 |
| 45943 | 2320 |
| 46584 | 2474 |
| 47240 | 2640 |
| 47849 | 2785 |
| 48488 | 2944 |
| 49193 | 3100 |
| 49823 | 3200 |
| 50481 | 3360 |
| 51150 | 3525 |
| 51799 | 3676 |
| 52393 | 3806 |
| 52976 | 3950 |
| 53667 | 4097 |

![BGPFS-eTCAM-Rate.png]({{site.baseurl}}/images/BGPFS-eTCAM-Rate.png){: .align-center}

The programming rate in this external TCAM bank is around 250 rules per second, at least in the boundaries of the supported scale (up to 3000).


## References

[Youtube video: Cisco NCS5500 Flowspec (Principles and Configuration) Part1](https://www.youtube.com/watch?v=dTgh0p9Vyns)
[https://www.youtube.com/watch?v=dTgh0p9Vyns](https://www.youtube.com/watch?v=dTgh0p9Vyns)

[Youtube video: BRKSPG 3012 - Leveraging BGP Flowspec to protect your infrastructure](https://www.youtube.com/watch?v=dbsNf8DcNRQ)
[https://www.youtube.com/watch?v=dbsNf8DcNRQ](https://www.youtube.com/watch?v=dbsNf8DcNRQ)

[Youtube video: Cisco NCS5500 Flowspec (Auto-Mitigation of a Memcached Attack) Part2](https://www.youtube.com/watch?v=iRPob7Ws2v8)
[https://www.youtube.com/watch?v=iRPob7Ws2v8](https://www.youtube.com/watch?v=iRPob7Ws2v8)

[SupportForum: BGP Flowspec implementation on NCS5500 platforms](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)
[https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)

[SupportForum: NCS5500 BGP flowspec packet matching criteria](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)
[https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)


## Conclusion/Acknowledgements

This post aimed at clarifying some specific aspects of the NCS550 BGP Flowspec implementation.  
- the space used by Flowspec rules is variable and dependent on the complexity
- ranges can use different memory sizes and it's usually the best to use power of twos
- the officially supported scale is 3000 "simple" rules
- the NCS55A2-MOD-SE-S based on Jericho+ with OP eTCAM can program up to 250 rules per second
- exceeding the scale won't have much consequences

We will update it with new content and corrections in the future if required.  
As usual, use the comment section below for your questions.

Thanks to Kirill Kasavchenko, Didier Urie and Ashok Kumar for their help and feedback.
