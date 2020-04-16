---
published: false
date: '2020-04-14 10:44 +0200'
title: NCS55A1-24H / NCS55A1-48Q6H Route Profiles Configuration
author: Nicolas Fevrier
excerpt: >-
  Some thoughts about the best hw-profile to best handle the internet size
  growth
tags:
  - large lpm
  - ncs5500
  - ios xr
position: hidden
---
{% include toc icon="table" title="NCS5500 Route Profiles for -24H and -48Q6H" %} 

You can find more content related to NCS5500 including routing memory management, URPF, ACLs, Netflow, eVPN, scale and other features following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

The NCS5500 portfolio is based on different DNX forwarding ASICs:  
- Jericho
- Qumran-MX
- Jericho+
	- BCM88683
    - BCM88680
- Jericho2

You notice we use different flavors of the Jericho+ NPU. One of the main difference between the two, the size of the LPM (Longest Prefix Match Database, sometimes referred to as KAPS for KBP Assisted Prefix Search, KBP being itself Knowledge Based Processor) allows to offer an intermediate routing scale between systems with and systems without external TCAM.  

![Screen Shot 2020-04-15 at 6.44.45 PM.png]({{site.baseurl}}/images/Screen Shot 2020-04-15 at 6.44.45 PM.png){: .align-center}

The platforms based on 88680 are the specific focus of this blog post:  
- NCS55A1-24H
- NCS55A1-48Q6H

![]({{site.baseurl}}/images/Screen%20Shot%202020-04-15%20at%206.44.56%20PM.png){: .align-center}

## hw-module profiles

We created a specific blog post on the many hardware profiles used to carve all the resources: 
[https://xrdocs.io/ncs5500/tutorials/ncs5500-hw-module-profiles/](https://xrdocs.io/ncs5500/tutorials/ncs5500-hw-module-profiles/)

And we already mentioned in another one ([https://xrdocs.io/ncs5500/tutorials/ncs5500-things-to-know/](https://xrdocs.io/ncs5500/tutorials/ncs5500-things-to-know/)) that we don't recommend to use the internet-optimized profile for the platforms based on Jericho+ with large LPM.

Now the question we want to address with this blog post is the following: should we keep the default "host-optimized" profile enabled, or will it be recommended in the longer run to disable this default mode via configuration to address the internet growth.

Before doing some projection on the growth, let's take a quick look at the current resource utilization with the two options, and also present a quick refresher on the way these profiles impact the route distribution between LEM and LPM.

### Internet feed used for the test

We recorded this internet full tables, v4 and v6, from a live network in Asia, in end of 2019.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh bgp sum

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 1512155
BGP main routing table version 1512155
BGP NSR Initial initsync version 241068 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         1512155    1512155    1512155    1512155     1512155           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100 1719113      12  1512155    0    0 00:09:20     <mark>790771</mark>

RP/0/RP0/CPU0:24H-1-701#sh bgp ipv6 un sum

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 72957
BGP main routing table version 72957
BGP NSR Initial initsync version 72956 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker           72957      72957      72957      72957       72957           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100   72961      12    72957    0    0 00:09:34      <mark>72949</mark>

RP/0/RP0/CPU0:24H-1-701#sh dpa resources iproute loc 0/0/CPU0

"iproute" OFA Table (Id: 37, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       1            /1       0
 /2       4            /3       5
 /4       1            /5       0
 /6       0            /7       0
 /8       10           /9       12
 /10      36           /11      97
 /12      285          /13      571
 /14      1143         /15      1913
 /16      13184        /17      7901
 /18      13534        /19      25210
 /20      39182        /21      47039
 /22      100821       /23      78898
 /24      445773       /25      144
 /26      211          /27      383
 /28      537          /29      721
 /30      3241         /31      440
 /32      9495

OFA Infra Stats Summary
                 Create Requests: 1605611
                 Delete Requests: 814819
                 Update Requests: 23

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0

                          NPU ID: NPU-0                  NPU-1
           Create Server API Err: 0                      0
           Update Server API Err: 0                      0
           Delete Server API Err: 0                      0

RP/0/RP0/CPU0:24H-1-701#sh dpa resources ip6route loc 0/0/CPU0

"ip6route" OFA Table (Id: 38, Scope: Global)
--------------------------------------------------
IPv6 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       1            /1       0
 /2       0            /3       0
 /4       0            /5       0
 /6       0            /7       0
 /8       0            /9       0
 /10      1            /11      0
 /12      0            /13      0
 /14      0            /15      0
 /16      4            /17      0
 /18      0            /19      2
 /20      12           /21      4
 /22      7            /23      5
 /24      23           /25      7
 /26      13           /27      18
 /28      94           /29      2696
 /30      418          /31      179
 /32      12686        /33      1060
 /34      814          /35      517
 /36      2887         /37      501
 /38      908          /39      282
 /40      3689         /41      544
 /42      888          /43      144
 /44      4720         /45      465
 /46      2223         /47      1352
 /48      35009        /49      0
 /50      0            /51      1
 /52      0            /53      0
 /54      0            /55      0
 /56      219          /57      2
 /58      0            /59      0
 /60      16           /61      0
 /62      0            /63      3
 /64      53           /65      0
 /66      0            /67      0
 /68      0            /69      0
 /70      0            /71      0
 /72      0            /73      0
 /74      0            /75      0
 /76      0            /77      0
 /78      0            /79      0
 /80      0            /81      0
 /82      0            /83      0
 /84      0            /85      0
 /86      0            /87      0
 /88      0            /89      0
 /90      0            /91      0
 /92      0            /93      0
 /94      0            /95      0
 /96      0            /97      0
 /98      0            /99      0
 /100     0            /101     0
 /102     0            /103     0
 /104     1            /105     0
 /106     0            /107     0
 /108     0            /109     0
 /110     0            /111     0
 /112     0            /113     0
 /114     0            /115     0
 /116     0            /117     0
 /118     0            /119     0
 /120     2            /121     0
 /122     0            /123     0
 /124     1            /125     8
 /126     432          /127     16
 /128     38

OFA Infra Stats Summary
                 Create Requests: 208986
                 Delete Requests: 136021
                 Update Requests: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0

                          NPU ID: NPU-0                  NPU-1
           Create Server API Err: 0                      0
           Update Server API Err: 0                      0
           Delete Server API Err: 0                      0

RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

### host-optimized mode enabled (default)

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh run | i hw

Building configuration...
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0

HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : <mark>490272</mark>   <mark>(62 %)</mark>
        iproute                     : <mark>455268</mark>   <mark>(58 %)</mark>
        ip6route                    : <mark>35009</mark>    <mark>(4 %)</mark>
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 490272   (62 %)
        iproute                     : 455268   (58 %)
        ip6route                    : 35009    (4 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lpm loc 0/0/CPU0

HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1565853
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : <mark>373479</mark>   <mark>(24 %)</mark>
        iproute                     : <mark>335526</mark>   <mark>(21 %)</mark>
        ip6route                    : <mark>37957</mark>    <mark>(2 %)</mark>
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 1565853
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 373479   (24 %)
        iproute                     : 335526   (21 %)
        ip6route                    : 37957    (2 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

| LEM Used | LPM Used | LPM Estimated Max |
|:-----|:-----:|:-----:|
| 490272 | 373479 | 1565853 |
| 62% | 24% | - |

In default mode, IPv4/24, IPv4/32 and IPv6/48 are stored in LEM, the rest will go in LPM.

![Screen Shot 2020-04-15 at 9.24.48 PM.png]({{site.baseurl}}/images/Screen Shot 2020-04-15 at 9.24.48 PM.png){: .align-center}

### host-optimized-disable

This mode needs to be configured (and a reload of the chassis is necessary to enable it).

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh run | i hw

Building configuration...
hw-module fib ipv4 scale host-optimized-disable

RP/0/RP0/CPU0:24H-1-701#sh contr npu resources all loc 0/0/CPU0

HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : <mark>44495</mark>    <mark>(6 %)</mark>
        iproute                     : <mark>9495</mark>     <mark>(1 %)</mark>
        ip6route                    : <mark>35009</mark>    <mark>(4 %)</mark>
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 44495    (6 %)
        iproute                     : 9495     (1 %)
        ip6route                    : 35009    (4 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1495107
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : <mark>712073</mark>   <mark>(48 %)</mark>
        iproute                     : <mark>674122</mark>   <mark>(45 %)</mark>
        ip6route                    : <mark>37956</mark>    <mark>(3 %)</mark>
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 1495107
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 712073   (48 %)
        iproute                     : 674122   (45 %)
        ip6route                    : 37956    (3 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

// SNIP //

RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

| LEM Used | LPM Used | LPM Estimated Max |
|:-----|:-----:|:-----:|
| 44495 | 712073 | 1495107 |
| 6% | 48% | - |

In this host-disable mode, only IPv4/32 and IPv6/48 are going to LEM.

![Screen Shot 2020-04-16 at 12.40.52 AM.png]({{site.baseurl}}/images/Screen Shot 2020-04-16 at 12.40.52 AM.png){: .align-center}

## Handling internet growth

For this exercise, I'll use one more time the twitter feed created by Darren O'Connor (@mellowdrifter): 
- [https://twitter.com/bgp4_table](https://twitter.com/bgp4_table)
- [https://twitter.com/bgp6_table](https://twitter.com/bgp6_table)

In 


