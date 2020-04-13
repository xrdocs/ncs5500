---
published: false
date: '2020-04-13 00:05 +0200'
title: 'NCS5500 FIB Scale Test [Lab Series 04] '
author: Nicolas Fevrier
excerpt: Testing full internet views and up to 4M prefixes
position: hidden
---
{% include toc icon="table" title="NCS5500 FIB Scale Tests" %} 

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Episode 04 of the lab series, today, we will talk about NCS5500 FIB. More specifically, we will verify the number of routes we can store in systems with Jericho+ and eTCAM.  
The goal of these blog posts and videos is to describe tests performed in lab, detail the methodology and the results.  
They are extracted from customers POC (proof of concept). With this information, we hope it will speed up your validation process and provide additional information on the NCS5500 platforms internals.  
All former tests are listed here: [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)

## Video

In the video below, we performing two tests on Jericho+ systems with external memory (eTCAM):
- how much space consumes a full internet view (both v4 and v6)?
- can we push 4M IPv4 prefixes in this memory?

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/oglYEDpKsLY?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## Demo

Since the video has been recorded a few months back, we will actually use a more fresher internet view collected from one of our asian customer. Then we will re-do the 4M IPv4 prefixes test.  
To complete these tests, we will also activate URFP loose mode on some interfaces and we will also activate BGP Flowspec and push 3000 rules.

### Full internet view

We start with a single view made of 790k IPv4 and 72 IPv6 routes.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh bgp sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 19451056
BGP main routing table version 19451056
BGP NSR Initial initsync version 240593 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker        19451056   19451056   19451056   19451056    19451056           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.152   0   100 1061397   24179 19451056    0    0 00:04:18     <mark>790769</mark>
192.168.100.153   0   100 4789584   24072        0    0    0 15:01:03 Idle (Admin)

RP/0/RP0/CPU0:OCSE-653#sh bgp ipv6 un sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 4364742
BGP main routing table version 4364742
BGP NSR Initial initsync version 72950 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         4364742    4364742    4364742    4364742     4364742           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100  242899   24068  4364742    0    0 00:01:10      <mark>72948</mark>
2001:111::152     0   100   30725   25124        0    0    0     4w2d Idle (Admin)

RP/0/RP0/CPU0:OCSE-653#sh dpa resources iproute loc 0/0/CPU0

"iproute" OFA Table (Id: 25, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       4            /1       0
 /2       4            /3       5
 /4       4            /5       0
 /6       0            /7       0
 /8       10           /9       12
 /10      36           /11      97
 /12      285          /13      571
 /14      1143         /15      1913
 /16      13186        /17      7901
 /18      13534        /19      25210
 /20      39182        /21      47039
 /22      100821       /23      78898
 /24      445789       /25      144
 /26      211          /27      383
 /28      537          /29      721
 /30      3241         /31      440
 /32      9547
                          NPU ID: NPU-0           NPU-1           NPU-2           NPU-3
                          In Use: 790868          790868          790868          790868
                 Create Requests
                           Total: 11358014        11358014        11358014        11358014
                         Success: 11358014        11358014        11358014        11358014
                 Delete Requests
                           Total: 10567146        10567146        10567146        10567146
                         Success: 10567146        10567146        10567146        10567146
                 Update Requests
                           Total: 838044          838044          838044          838044
                         Success: 837906          837906          837906          837906
                    EOD Requests
                           Total: 0               0               0               0
                         Success: 0               0               0               0
                          Errors
                     HW Failures: 0               0               0               0
                Resolve Failures: 0               0               0               0
                 No memory in DB: 0               0               0               0
                 Not found in DB: 0               0               0               0
                    Exists in DB: 0               0               0               0
      Reserve Resources Failures: 0               0               0               0
      Release Resources Failures: 0               0               0               0
       Update Resources Failures: 0               0               0               0

RP/0/RP0/CPU0:OCSE-653#sh dpa resources ip6route loc 0/0/CPU0

"ip6route" OFA Table (Id: 26, Scope: Global)
--------------------------------------------------
IPv6 Prefix len distribution
Prefix   Actual            Capacity    Prefix   Actual            Capacity
 /0       4                 0            /1       0                 0
 /2       0                 0            /3       0                 0
 /4       0                 0            /5       0                 0
 /6       0                 0            /7       0                 0
 /8       0                 0            /9       0                 0
 /10      4                 0            /11      0                 0
 /12      0                 0            /13      0                 0
 /14      0                 0            /15      0                 0
 /16      13                0            /17      0                 0
 /18      0                 0            /19      2                 0
 /20      12                0            /21      4                 0
 /22      7                 0            /23      5                 0
 /24      23                0            /25      7                 0
 /26      13                0            /27      18                0
 /28      94                0            /29      2696              0
 /30      418               0            /31      179               0
 /32      12686             0            /33      1060              0
 /34      814               0            /35      517               0
 /36      2887              0            /37      501               0
 /38      908               0            /39      282               0
 /40      3689              0            /41      544               0
 /42      888               0            /43      144               0
 /44      4720              0            /45      465               0
 /46      2223              0            /47      1352              0
 /48      35009             0            /49      0                 0
 /50      0                 0            /51      1                 0
 /52      0                 0            /53      0                 0
 /54      0                 0            /55      0                 0
 /56      219               0            /57      2                 0
 /58      0                 0            /59      0                 0
 /60      16                0            /61      0                 0
 /62      0                 0            /63      3                 0
 /64      67                0            /65      0                 0
 /66      0                 0            /67      0                 0
 /68      0                 0            /69      0                 0
 /70      0                 0            /71      0                 0
 /72      0                 0            /73      0                 0
 /74      0                 0            /75      0                 0
 /76      0                 0            /77      0                 0
 /78      0                 0            /79      0                 0
 /80      0                 0            /81      0                 0
 /82      0                 0            /83      0                 0
 /84      0                 0            /85      0                 0
 /86      0                 0            /87      0                 0
 /88      0                 0            /89      0                 0
 /90      0                 0            /91      0                 0
 /92      0                 0            /93      0                 0
 /94      0                 0            /95      0                 0
 /96      0                 0            /97      0                 0
 /98      0                 0            /99      0                 0
 /100     0                 0            /101     0                 0
 /102     0                 0            /103     0                 0
 /104     4                 0            /105     0                 0
 /106     0                 0            /107     0                 0
 /108     0                 0            /109     0                 0
 /110     0                 0            /111     0                 0
 /112     0                 0            /113     0                 0
 /114     0                 0            /115     0                 0
 /116     0                 0            /117     0                 0
 /118     0                 0            /119     0                 0
 /120     2                 0            /121     0                 0
 /122     0                 0            /123     0                 0
 /124     1                 0            /125     8                 0
 /126     432               0            /127     16                0
 /128     55                0
                          NPU ID: NPU-0           NPU-1           NPU-2           NPU-3
                          In Use: 73014           73014           73014           73014
                 Create Requests
                           Total: 2220292         2220292         2220292         2220292
                         Success: 2220292         2220292         2220292         2220292
                 Delete Requests
                           Total: 2147278         2147278         2147278         2147278
                         Success: 2147278         2147278         2147278         2147278
                 Update Requests
                           Total: 266             266             266             266
                         Success: 202             202             202             202
                    EOD Requests
                           Total: 0               0               0               0
                         Success: 0               0               0               0
                          Errors
                     HW Failures: 0               0               0               0
                Resolve Failures: 0               0               0               0
                 No memory in DB: 0               0               0               0
                 Not found in DB: 0               0               0               0
                    Exists in DB: 0               0               0               0
      Reserve Resources Failures: 0               0               0               0
      Release Resources Failures: 0               0               0               0
       Update Resources Failures: 0               0               0               0

RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv4 loc 0/0/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv4

OOR Information
    NPU-0
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:27 UTC
    NPU-1
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:25 UTC
    NPU-2
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:25 UTC
    NPU-3
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:26 UTC

Current Usage
    NPU-0
        Total In-Use                : <mark>790856</mark>   <mark>(20 %)</mark>
        iproute                     : 790868   (20 %)
    NPU-1
        Total In-Use                : 790856   (20 %)
        iproute                     : 790868   (20 %)
    NPU-2
        Total In-Use                : 790856   (20 %)
        iproute                     : 790868   (20 %)
    NPU-3
        Total In-Use                : 790856   (20 %)
        iproute                     : 790868   (20 %)

RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv6 loc 0/0/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv6

OOR Information
    NPU-0
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:27 UTC
    NPU-1
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:25 UTC
    NPU-2
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:25 UTC
    NPU-3
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green
        OOR State Change Time       : 2020.Apr.12 23:19:26 UTC

Current Usage
    NPU-0
        Total In-Use                : <mark>72990</mark>    <mark>(4 %)</mark>
        ip6route                    : 73014    (4 %)
    NPU-1
        Total In-Use                : 72990    (4 %)
        ip6route                    : 73014    (4 %)
    NPU-2
        Total In-Use                : 72990    (4 %)
        ip6route                    : 73014    (4 %)
    NPU-3
        Total In-Use                : 72990    (4 %)
        ip6route                    : 73014    (4 %)

RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

It clearly shows the systems based on Jericho+ with eTCAM have a ton of memory space when used for internet peering / border roles.

### 4M IPv4 routes

On this one, we are using a route generator with extremely basic configuration. The goal being to use the route memory to the fullest supported level: 4 millions IPv4 entries.

<div class="highlighter-rouge">
<pre class="highlight">
<code>router bgp 100
bgp_id 192.168.100.153
neighbor 192.168.100.202 remote-as 100
neighbor 192.168.100.202 update-source 192.168.100.152
capability ipv4 unicast
capability refresh
network 1 1.1.1.1/32 4000000
aspath 1 random 5
locpref 1 120
metric 1 5
sendall</code>
</pre>
</div>

On the router, we see the routes received and programmed:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh bgp sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 14558159
BGP main routing table version 14558159
BGP NSR Initial initsync version 240593 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker        14558159   14558159   14558159   14558159    14558159           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.152   0   100   31280   24091 14558159    0    0 00:41:12    4000000
192.168.100.153   0   100 4789584   24072        0    0    0 00:43:26 Idle (Admin)

RP/0/RP0/CPU0:OCSE-653#sh dpa resources iproute loc 0/0/CPU0

"iproute" OFA Table (Id: 25, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       4            /1       0
 /2       0            /3       0
 /4       4            /5       0
 /6       0            /7       0
 /8       0            /9       0
 /10      0            /11      0
 /12      0            /13      0
 /14      0            /15      0
 /16      2            /17      0
 /18      0            /19      0
 /20      0            /21      0
 /22      0            /23      0
 /24      20           /25      0
 /26      0            /27      0
 /28      0            /29      0
 /30      0            /31      0
 /32      4000068
                          NPU ID: NPU-0           NPU-1           NPU-2           NPU-3
                          In Use: 4000098         4000098         4000098         4000098
                 Create Requests
                           Total: 10567244        10567244        10567244        10567244
                         Success: 10567244        10567244        10567244        10567244
                 Delete Requests
                           Total: 6567146         6567146         6567146         6567146
                         Success: 6567146         6567146         6567146         6567146
                 Update Requests
                           Total: 838040          838040          838040          838040
                         Success: 837903          837903          837903          837903
                    EOD Requests
                           Total: 0               0               0               0
                         Success: 0               0               0               0
                          Errors
                     HW Failures: 0               0               0               0
                Resolve Failures: 0               0               0               0
                 No memory in DB: 0               0               0               0
                 Not found in DB: 0               0               0               0
                    Exists in DB: 0               0               0               0
      Reserve Resources Failures: 0               0               0               0
      Release Resources Failures: 0               0               0               0
       Update Resources Failures: 0               0               0               0

RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv4 loc 0/0/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv4

OOR Information
    NPU-0
        Estimated Max Entries       : 4000086
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.12 21:21:33 UTC
    NPU-1
        Estimated Max Entries       : 4000086
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.12 21:21:33 UTC
    NPU-2
        Estimated Max Entries       : 4000086
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.12 21:21:33 UTC
    NPU-3
        Estimated Max Entries       : 4000086
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.12 21:21:33 UTC

Current Usage
    NPU-0
        Total In-Use                : 4000086  (100 %)
        iproute                     : 4000098  (100 %)
    NPU-1
        Total In-Use                : 4000086  (100 %)
        iproute                     : 4000098  (100 %)
    NPU-2
        Total In-Use                : 4000086  (100 %)
        iproute                     : 4000098  (100 %)
    NPU-3
        Total In-Use                : 4000086  (100 %)
        iproute                     : 4000098  (100 %)

RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

