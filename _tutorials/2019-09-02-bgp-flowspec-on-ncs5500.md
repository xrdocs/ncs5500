---
published: false
date: '2019-09-02 10:29 +0200'
title: BGP FlowSpec on NCS5500
position: hidden
author: Nicolas Fevrier
excerpt: BGP Flowspec implementation and resource management on NCS5500 Series
---
{% include toc icon="table" title="BGP FlowSpec on NCS5500" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Yosef already publish around BGP FlowSpec and more particular its implementation on the NCS5500 routers on Cisco's SupportForum:  
- [SupportForum: BGP Flowspec implementation on NCS5500 platforms](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443): [https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)
- [SupportForum: NCS5500 BGP flowspec packet matching criteria](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443): [https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443](https://community.cisco.com/t5/service-providers-blogs/bgp-flowspec-implementation-on-ncs5500-platforms/ba-p/3387443)

Today, we will dig a bit deeper in the subtleties of this implementation, presenting the memory spaces used to store the rules information and the statistics, and running a couple of tests to identify the limits.

First reminder: the support is limited today (September 2019) to the platforms based on Jericho+ NPU and External TCAM.  
Namely:  
- NCS55A1-36H-SE-S
- NCS55A2-MOD-SE-S (the one we are using for these tests)
- NC55-36X100G-A-SE line card (36x100G -SE)
- NC55-MOD-A-SE-S line card (MOD -SE)

Here are three videos on Youtube that could answer most of your questions on the topic:

The first one covers all the principles and details the configuration steps:  
[Cisco NCS5500 Flowspec (Principles and Configuration) Part1](https://www.youtube.com/watch?v=dTgh0p9Vyns)  
<iframe width="560" height="315" src="https://www.youtube.com/embed/dTgh0p9Vyns" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

Then a simple example of interoperability demo between Arbor SP products and NCS5500 to automitigate a common attack vector such as an MemCacheD amplification attack:  
[Cisco NCS5500 Flowspec (Auto-Mitigation of a Memcached Attack) Part2](https://www.youtube.com/watch?v=iRPob7Ws2v8)  
<iframe width="560" height="315" src="https://www.youtube.com/embed/iRPob7Ws2v8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

Finally, the CiscoLive session dedicated to BGP FlowSpec. A deepdive in the technology:  
[https://www.youtube.com/watch?v=dbsNf8DcNRQ](https://www.youtube.com/watch?v=dbsNf8DcNRQ)  
[BRKSPG 3012 - Leveraging BGP Flowspec to protect your infrastructure](https://www.youtube.com/watch?v=dbsNf8DcNRQ)  





## Specific NCS5500 implementation

Recirculation

IPv6 requires a specific mode, with lower PPS


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
!</code>
</pre>
</div>


## Tests

### Advertisement 4000 rules and verification of the resources consumed

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh bgp ipv4 flowspec sum

BGP router identifier 1.1.1.111, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 6004
BGP main routing table version 6004
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker            6004       6004       6004       6004        6004           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100      68      34     6004    0    0 00:00:15       4000

RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>



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

--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--SNIP--
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
                          In Use: <mark>4000</mark>
                 Create Requests
                           Total: 9000
                         Success: 9000
                 Delete Requests
                           Total: 5000
                         Success: 5000
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

The BGP FlowSpec rules are stored in external TCAM:

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
0    11     320b   FLP         96       <mark>4000</mark>    126  <mark>INGRESS_FLOWSPEC_IPV4</mark>
RP/0/RP0/CPU0:Peyto-SE#</code>
</pre>
</div>

Nothing is used in the other most common resources: LPM, LEM, IPv4 eTCAM, EEDB, FEC/ECMPFEC or iTCAM

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Peyto-SE#sh contr npu resources all loc 0/0/CPU0
Sun Jul 14 15:04:36.090 UTC
HW Resource Information
    Name                            : lem

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>0</mark>        <mark>(0 %)</mark>
        iproute                     : 0        (0 %)
        ip6route                    : 0        (0 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 329283
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>4</mark>        <mark>(0 %)</mark>
        iproute                     : 0        (0 %)
        ip6route                    : 0        (0 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

HW Resource Information
    Name                            : encap

OOR Information
    NPU-0
        Estimated Max Entries       : 104000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>0</mark>        <mark>(0 %)</mark>
        ipnh                        : 0        (0 %)
        ip6nh                       : 0        (0 %)
        mplsnh                      : 0        (0 %)

HW Resource Information
    Name                            : ext_tcam_ipv4

OOR Information
    NPU-0
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>6</mark>        <mark>(0 %)</mark>
        iproute                     : 9        (0 %)

HW Resource Information
    Name                            : fec

OOR Information
    NPU-0
        Estimated Max Entries       : 126976
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>15</mark>       <mark>(0 %)</mark>
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
    Name                            : ecmp_fec

OOR Information
    NPU-0
        Estimated Max Entries       : 4096
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>0</mark>        <mark>(0 %)</mark>
        ipnhgroup                   : 0        (0 %)
        ip6nhgroup                  : 0        (0 %)

HW Resource Information
    Name                            : ext_tcam_ipv6

OOR Information
    NPU-0
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : <mark>3</mark>        <mark>(0 %)</mark>
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

On the statistics side:

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
    ACL RX, LPTS               <mark>915</mark>     915  |     ACL RX, LPTS               915     915
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
    ACL RX, LPTS              <mark>3288</mark>    8192  |     ACL RX, LPTS              3288    8192
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

### Check behavior with more than 4096 rules



### Session limit configuration



### Verification of the resource used with complex rules (with ranges)



### Programming rate



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

------- Template -------


Blablabla

**Update**: blablabla
{: .notice--info}

![Base vs Scale]({{site.baseurl}}/images/base-scale.jpg){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:Router#sh platform | i XR RUN <mark>NCS-5501</mark>
RP/0/RP0/CPU0:Router#</code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

__500k IPv4 /24 routes__

In this second example, we announce 500,000 IPv4/24 prefixes.
With both **host-optimized** and **internet-optimized** profiles on base line cards, we will see these prefixes moved into LEM.

| CDN / Content Providers | Ingress Avg | Egress Avg |
|:------:|:------:|:-------:|
| Cust 1 Google GGC | 1277 | 329 |
| Cust 2 Google GGC | 1370 | 350 |
| Cust 3 Google GGC | 1393 | 284 |

We recommend to start watching this short Youtube video first:  
[https://www.youtube.com/watch?v=1qXD70_cLK8](https://www.youtube.com/watch?v=1qXD70_cLK8)

<iframe width="560" height="315" src="https://www.youtube.com/embed/1qXD70_cLK8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


