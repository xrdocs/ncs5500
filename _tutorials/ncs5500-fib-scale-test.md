---
published: true
date: '2020-04-13 00:05 +0200'
title: 'NCS5500 FIB Scale Test [Lab Series 04] '
author: Nicolas Fevrier
excerpt: Testing full internet views and up to 4M prefixes in J+/eTCAM NCS5500 systems
position: top
tags:
  - lab series
  - ncs5500
  - ios xr
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

We reached an OOR (out of resource) state "Red" since we exceeded 95% of the "Estimated Max Entries" capacity, but the routes are programmed successfully in hardware.  
You can verify it with the "HW Failures:" counters in the "show dpa resource" output.  

### 2M IPv6 routes

Let's see if we can push a lot of IPv6 routes too.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh bgp ipv6 un sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 26437690
BGP main routing table version 26437690
BGP NSR Initial initsync version 72950 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker        26437690   26437690   26437690   26437690    26437690           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100  279980   24234 26437690    0    0 00:03:05    <mark>2000000</mark>
2001:111::152     0   100   30725   25124        0    0    0     4w2d Idle (Admin)

RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv6 loc 0/0/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv6

OOR Information
    NPU-0
        Estimated Max Entries       : 2000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:35:32 PDT
    NPU-1
        Estimated Max Entries       : 2000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:35:32 PDT
    NPU-2
        Estimated Max Entries       : 2000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:35:32 PDT
    NPU-3
        Estimated Max Entries       : 2000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:35:32 PDT

Current Usage
    NPU-0
        Total In-Use                : <mark>2000040</mark>  <mark>(100 %)</mark>
        ip6route                    : 2000064  (100 %)
    NPU-1
        Total In-Use                : 2000040  (100 %)
        ip6route                    : 2000064  (100 %)
    NPU-2
        Total In-Use                : 2000040  (100 %)
        ip6route                    : 2000064  (100 %)
    NPU-3
        Total In-Use                : 2000040  (100 %)
        ip6route                    : 2000064  (100 %)

RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

As advertised, tested and officially supported, no problem pushing 2M IPv6 routes in the external TCAM.  
Of course, we are in Red OOR state but this limit of 2M is actually not hard coded in the system. We can potentially go further if needed. But as mentioned, it's not tested officially.  

Example with 3M IPv6/64 routes:  

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh bgp ipv6 un sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 31437690
BGP main routing table version 31437690
BGP NSR Initial initsync version 72950 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker        31437690   31437690   31437690   31437690    31437690           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100  290819   24242 31437690    0    0 00:01:37    <mark>3000000</mark>
2001:111::152     0   100   30725   25124        0    0    0     4w2d Idle (Admin)

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
 /16      12                0            /17      0                 0
 /18      0                 0            /19      0                 0
 /20      0                 0            /21      0                 0
 /22      0                 0            /23      0                 0
 /24      0                 0            /25      0                 0
 /26      0                 0            /27      0                 0
 /28      0                 0            /29      0                 0
 /30      0                 0            /31      0                 0
 /32      0                 0            /33      0                 0
 /34      0                 0            /35      0                 0
 /36      0                 0            /37      0                 0
 /38      0                 0            /39      0                 0
 /40      0                 0            /41      0                 0
 /42      0                 0            /43      0                 0
 /44      0                 0            /45      0                 0
 /46      0                 0            /47      0                 0
 /48      0                 0            /49      0                 0
 /50      0                 0            /51      0                 0
 /52      0                 0            /53      0                 0
 /54      0                 0            /55      0                 0
 /56      0                 0            /57      0                 0
 /58      0                 0            /59      0                 0
 /60      0                 0            /61      0                 0
 /62      0                 0            /63      0                 0
 <mark>/64      3000018</mark>           0            /65      0                 0
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
 /120     0                 0            /121     0                 0
 /122     0                 0            /123     0                 0
 /124     0                 0            /125     0                 0
 /126     0                 0            /127     0                 0
 /128     22                0
                          NPU ID: NPU-0           NPU-1           NPU-2           NPU-3
                          In Use: <mark>3000064</mark>         3000064         3000064         3000064
                 Create Requests
                           Total: 9958976         9958976         9958976         9958976
                         Success: 9958976         9958976         9958976         9958976
                 Delete Requests
                           Total: 6958912         6958912         6958912         6958912
                         Success: 6958912         6958912         6958912         6958912
                 Update Requests
                           Total: 0               0               0               0
                         Success: 0               0               0               0
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

RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv6 loc 0/0/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv6

OOR Information
    NPU-0
        Estimated Max Entries       : <mark>3000040</mark>
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:42:21 PDT
    NPU-1
        Estimated Max Entries       : 3000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:42:21 PDT
    NPU-2
        Estimated Max Entries       : 3000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:42:21 PDT
    NPU-3
        Estimated Max Entries       : 3000040
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Apr.13 08:42:21 PDT

Current Usage
    NPU-0
        Total In-Use                : <mark>3000040</mark>  <mark>(100 %)</mark>
        ip6route                    : 3000064  (100 %)
    NPU-1
        Total In-Use                : 3000040  (100 %)
        ip6route                    : 3000064  (100 %)
    NPU-2
        Total In-Use                : 3000040  (100 %)
        ip6route                    : 3000064  (100 %)
    NPU-3
        Total In-Use                : 3000040  (100 %)
        ip6route                    : 3000064  (100 %)

RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

### Other features impact: BGP Flowspec

Let's start with the 4M IPv4 routes.
Flowspec is configured but we don't receive any rule at the moment (session is not active).

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh run router bgp 100 neighbor 192.168.100.151
router bgp 100
 neighbor 192.168.100.151
  remote-as 100
  address-family ipv4 flowspec
   route-policy PERMIT-ANY in
   maximum-prefix 8000000 75
   route-policy PERMIT-ANY out
  !
 !
!

RP/0/RP0/CPU0:OCSE-653#sh run flowspec
flowspec
 local-install interface-all
 address-family ipv4
  service-policy type pbr scale_ipv4
 !
!

RP/0/RP0/CPU0:OCSE-653#sh bgp ipv4 flowspec sum

BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 202
BGP main routing table version 202
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker             202        202        202        202         202           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100       5       5        0    0    0 00:11:57 <mark>Active</mark>

RP/0/RP0/CPU0:OCSE-653#sh contr npu externaltcam loc 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         2871968  <mark>4000086</mark>  0    <mark>IPv4 UC</mark>
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         713193   39      3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
0    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
0    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
0    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
0    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
0    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
0    10     320b   FLP         4094     <mark>2</mark>       118  <mark>INGRESS_FLOWSPEC_IPV4</mark>
1    0      80b    FLP         2871968  4000086  0    IPv4 UC
1    1      80b    FLP         0        0       1    IPv4 RPF
1    2      160b   FLP         713193   39      3    IPv6 UC
1    3      160b   FLP         0        0       4    IPv6 RPF
1    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
1    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
1    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
1    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
1    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
1    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
1    10     320b   FLP         4094     2       118  INGRESS_FLOWSPEC_IPV4
2    0      80b    FLP         2871968  4000086  0    IPv4 UC
2    1      80b    FLP         0        0       1    IPv4 RPF
2    2      160b   FLP         713193   39      3    IPv6 UC
2    3      160b   FLP         0        0       4    IPv6 RPF
2    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
2    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
2    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
2    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
2    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
2    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
2    10     320b   FLP         4094     2       118  INGRESS_FLOWSPEC_IPV4
3    0      80b    FLP         2871968  4000086  0    IPv4 UC
3    1      80b    FLP         0        0       1    IPv4 RPF
3    2      160b   FLP         713193   39      3    IPv6 UC
3    3      160b   FLP         0        0       4    IPv6 RPF
3    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
3    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
3    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
3    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
3    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
3    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
3    10     320b   FLP         4094     2       118  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

Now we advertise 3000 simple rules.

On the testing tool:

<div class="highlighter-rouge">
<pre class="highlight">
<code>router bgp 100
bgp_id 192.168.100.151
neighbor 192.168.100.202 remote-as 100
neighbor 192.168.100.202 update-source 192.168.100.151
capability ipv4 flowspec
network 1 ipv4 flowspec
network 1 dest 7.7.7.7/32 protocol 17 source-port 123
network 1 count 3000 dest-incr</code>
</pre>
</div>

On the router:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh bgp ipv4 flowspec sum
BGP router identifier 1.3.5.99, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0   RD version: 3402
BGP main routing table version 3402
BGP NSR Initial initsync version 0 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker            3402        402       3402       3402         402           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100      28      13      402    0    0 00:00:06       <mark>3000</mark>

RP/0/RP0/CPU0:OCSE-653#sh flowspec ipv4

AFI: IPv4
  Flow           :Dest:7.7.7.7/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.8/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.9/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.10/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.11/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.12/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.13/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.14/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.15/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.16/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)
  Flow           :Dest:7.7.7.17/32,Proto:=17,SPort:=123
    Actions      :transmit  (bgp.1)

///-- SNIP SNIP SNIP --//

RP/0/RP0/CPU0:OCSE-653#sh dpa resources ippbr loc 0/0/CPU0

"ippbr" OFA Table (Id: 137, Scope: Global)
--------------------------------------------------
                          NPU ID: NPU-0           NPU-1           NPU-2           NPU-3
                          In Use: <mark>3001</mark>            3001            3001            3001
                 Create Requests
                           Total: 3202            3202            3202            3202
                         Success: 3202            3202            3202            3202
                 Delete Requests
                           Total: 201             201             201             201
                         Success: 201             201             201             201
                 Update Requests
                           Total: 0               0               0               0
                         Success: 0               0               0               0
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

RP/0/RP0/CPU0:OCSE-653#sh contr npu externaltcam loc 0/0/CPU0

External TCAM Resource Information
=============================================================
NPU  Bank   Entry  Owner       Free     Per-DB  DB   DB
     Id     Size               Entries  Entry   ID   Name
=============================================================
0    0      80b    FLP         2871968  4000086  0    IPv4 UC
0    1      80b    FLP         0        0       1    IPv4 RPF
0    2      160b   FLP         713193   39      3    IPv6 UC
0    3      160b   FLP         0        0       4    IPv6 RPF
0    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
0    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
0    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
0    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
0    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
0    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
0    10     320b   FLP         1094     <mark>3002</mark>    118  <mark>INGRESS_FLOWSPEC_IPV4</mark>
1    0      80b    FLP         2871968  4000086  0    IPv4 UC
1    1      80b    FLP         0        0       1    IPv4 RPF
1    2      160b   FLP         713193   39      3    IPv6 UC
1    3      160b   FLP         0        0       4    IPv6 RPF
1    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
1    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
1    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
1    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
1    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
1    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
1    10     320b   FLP         1094     3002    118  INGRESS_FLOWSPEC_IPV4
2    0      80b    FLP         2871968  4000086  0    IPv4 UC
2    1      80b    FLP         0        0       1    IPv4 RPF
2    2      160b   FLP         713193   39      3    IPv6 UC
2    3      160b   FLP         0        0       4    IPv6 RPF
2    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
2    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
2    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
2    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
2    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
2    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
2    10     320b   FLP         1094     3002    118  INGRESS_FLOWSPEC_IPV4
3    0      80b    FLP         2871968  4000086  0    IPv4 UC
3    1      80b    FLP         0        0       1    IPv4 RPF
3    2      160b   FLP         713193   39      3    IPv6 UC
3    3      160b   FLP         0        0       4    IPv6 RPF
3    4      80b    FLP         4096     0       75   INGRESS_IPV4_SRC_IP_EXT
3    5      80b    FLP         4096     0       76   INGRESS_IPV4_DST_IP_EXT
3    6      160b   FLP         4096     0       77   INGRESS_IPV6_SRC_IP_EXT
3    7      160b   FLP         4096     0       78   INGRESS_IPV6_DST_IP_EXT
3    8      80b    FLP         4096     0       79   INGRESS_IP_SRC_PORT_EXT
3    9      80b    FLP         4096     0       80   INGRESS_IPV6_SRC_PORT_EXT
3    10     320b   FLP         1094     3002    118  INGRESS_FLOWSPEC_IPV4
RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

BGP Flowspec entries are stored in a different zone in the external TCAM then the part used for routing information.  

**Conclusion: BGP Flowspec doesn't impact the routing scale**

It would be an interesting study to quantify the impact of the complexity of the flowspec rule on the number of entries we can store in eTCAM. For the example above, we used simple rules, but other parameters like packet length ranges can consume more entries.
{: .notice-info}

### Other features impact: URPF

Let's try another feature and identify the impact on routing scale: Unicast Reverse Path Forwarding.  
To configure it on systems with Jericho+ and eTCAM, you don't need to enable any specific hw-module profile, which is not the case for other types of NCS5500. More details in:  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/](https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/)

We apply URPF on two interfaces and it does not have any impact.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:OCSE-653#sh run int hu0/0/0/0
interface HundredGigE0/0/0/0
 description OCSE H0/0/0/0 to 5508 H0/0/0/1
 cdp
 mtu 9646
 ipv4 address 25.1.11.2 255.255.255.0
 ipv4 verify unicast source reachable-via any
 ipv6 address 2001:25:1:11::2/64
 load-interval 30
!
RP/0/RP0/CPU0:OCSE-653#sh run int hu0/0/0/4
interface HundredGigE0/0/0/4
 description OCSE H0/0/0/4 to 24H H0/0/0/4
 cdp
 ipv4 address 25.1.110.2 255.255.255.0
 ipv4 verify unicast source reachable-via any
 ipv6 address 2001:25:1:110::2/64
 load-interval 30
!
RP/0/RP0/CPU0:OCSE-653#</code>
</pre>
</div>

No impact on the 4M routes.

## Conclusion

Very basic tests but regularly requested in the Customer Proof of Concept.  
We demonstrated that we can store a full internet feed IPv4 and IPv6 with a lot of growth margin, but also that we could store 4M IPv4 entries in the eTCAM while enabling other features like BGP Flowspec and URPF (loose mode) without any issue.

## What's next?

Next blog post, we still test the performance of the new line cards based on Jericho2 ASICs.  
If you would like specific tests executed in this series, please let us know in the comments below.
