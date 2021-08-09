---
published: true
date: '2021-08-04 22:56 +0200'
title: Full Internet in Jericho2 NPU and Programming Speed
author: Nicolas Fevrier
position: top
excerpt: Understand the J2 memory utilization and the programming / convergence speed
---
{% include toc icon="table" title="Full Internet in J2" %}  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/-wGhup-ZTZ4" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

Articles mentioned in the video:

- Decommissionning the internet-optimized mode:  
[https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/](https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/)  
- NCS5500 Routing Resource with 2020 Internet (and Future)  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/](https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/)  
- NCS5500 FIB Programming Speed  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/](https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/)  

## NCS5700 and the Jericho2 NPU

The NCS5500 products series is composed of different routing devices, fixed port "pizza boxes" and modular chassis with line cards. To clearly identify the products based on Broadcom Jericho2 ASICs, we named the fixed platforms "NCS-57xx" and the line cards in the chassis "NC57-xx". It's also the case for J2C platforms.  
The suffix "-SE" is used to describe systems and line cards with external TCAM:

| PID | Port Density | NPU | eTCAM |
|:-----:|:-----:|:-----:|:-----:|
| NCS57B1-6D24 | 24x 100GE, 6x 400GE | 1x J2 | No |
| NCS57B1-5DSE | 24x 100GE, 5x 400GE | 1x J2 | Yes |
| NC57-24DD | 24x 400GE | 2x J2 | No |
| NC57-18DD-SE | 24x Flex Ports + 6x 400G | 2x J2 | Yes |
| NC57-36H6D-S | 36x Flex Ports | 1x J2 | No |
| NC57-36H-SE | 36x 100GE | 1x J2 | Yes |

Note: More platforms coming soon in IOS XR 7.4.1 and in following releases
{: .notice--danger}

### Internal Resources

Like their predecessors, the J2 NPUs are leveraging different internal memories to store information, including routing details.

If not equipped with eTCAM, we have LEM and LPM:

![J2.png]({{site.baseurl}}/images/J2.png){: .align-center}

If the NPU is completed by an external TCAM:

![J2-SE.png]({{site.baseurl}}/images/J2-SE.png){: .align-center}

In next releases (roadmap at the moment of the redaction of this article), different MDB profiles will be available and the size of LEM and LPM may vary, but the logic used to store routing information will very likely stay the same. Of course, we will document all future deviation.

And essentially, things are very simple:  
- in J2 without eTCAM, all IPv4 and IPv6 prefixes are stored in LPM
- in J2 with eTCAM, all IPv4 and IPv6 prefixes are stored in... eTCAM

**_Important note:_**

A display bug is present in IOS XR 7.3.x and 7.4.x, DDTS CSCvw55441 titled "J2-non SE : "show contr npu resources" discrepancy for iproute with v4/32 routes"

This defect only impacts the systems with no eTCAM where the v4/v6 prefixes are all stored in LPM. It can create a confusion counting the v4/32 routes as stored in LEM which is not the case in reality, example to prove it with a prefix 1.1.1.191/32

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-741#sh route 1.1.1.191

Routing entry for 1.1.1.191/32
  Known via "local", distance 0, metric 0 (connected)
  Installed Jun 14 02:53:43.044 for 01:11:51
  Routing Descriptor Blocks
    directly connected, via Loopback0
      Route metric is 0
  Redist Advertisers:
    9 (protoid=9, clientid=33)
    5 (protoid=5, clientid=25)

RP/0/RP1/CPU0:5508-1-741#show  controllers fia diagshell 0 "dbal table dump table=IPV4_UNICAST_PRIVATE_LEM_FORWARD" location 0/7/CPU0 | i 1.1.1.191

RP/0/RP1/CPU0:5508-1-741#show  controllers fia diagshell 0 "dbal table dump table=IPV4_UNICAST_PRIVATE_LPM_FORWARD" location 0/7/CPU0 | i 1.1.1.191

| 31  | 0 mask: 0xffff | 1.1.1.191 (0x10101bf) mask: 0xffffffff        || KAPS_FLOW_ID 19 (655379)   |
RP/0/RP1/CPU0:5508-1-741#</code>
</pre>
</div>

Don't use this shell command in production since it dumps the entirety of the table. It's only meant to be used in the lab with small routing table, to verify where routes are actually going.

Until this DDTS is fixed, take the output of "show controller npu resources" with a grain of salt (or do the math yourself, counting the /32s from the DPA and adjusting the counters accordingly).

## Full internet view in Jericho2 platforms / LCs

We will run the test on a chassis equipped with a J2 line cards: non eTCAM (NC57-24DD) and with eTCAM (NC57-36H-SE):

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh ver

Cisco IOS XR Software, Version 7.3.1
Copyright (c) 2013-2021 by Cisco Systems, Inc.

Build Information:
 Built By     : ingunawa
 Built On     : Thu Feb 25 19:43:35 PST 2021
 Built Host   : iox-ucs-023
 Workspace    : /auto/srcarchive17/prod/7.3.1/ncs5500/ws
 Version      : 7.3.1
 Location     : /opt/cisco/XR/packages/
 Label        : 7.3.1

cisco NCS-5500 () processor
System uptime is 16 hours 8 minutes

RP/0/RP1/CPU0:5508-1-731#sh platform | i IOS XR

0/0/CPU0          NC55-36X100G-A-SE          IOS XR RUN        NSHUT
0/3/CPU0          NC57-36H-SE                IOS XR RUN        NSHUT
0/7/CPU0          NC57-24DD                  IOS XR RUN        NSHUT
0/RP1/CPU0        NC55-RP-E(Active)          IOS XR RUN        NSHUT
RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

We receive a full table v4 and v6 from a single peer (each):

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh bgp sum

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 6144813
BGP main routing table version 6144813
BGP NSR Initial initsync version 7 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         6144813    6144813    6144813    6144813     6144813           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
25.2.19.2         0   100     968  282819  6144813    0    0 16:05:07          0
192.168.100.151   0  1000  836808  172278  6144813    0    0 00:40:30     836741
192.168.100.152   0  1000  691695     626        0    0    0 00:53:00 Active
192.168.100.153   0  1000    1494  118138        0    0    0 00:03:04 Active
192.168.100.154   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.155   0  1000     753     754        0    0    0 13:36:26 Active
192.168.100.156   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.157   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.158   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.159   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.160   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.161   0   100       0       0        0    0    0 00:00:00 Active

RP/0/RP1/CPU0:5508-1-731#sh bgp ipv6 un sum

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 599762
BGP main routing table version 599762
BGP NSR Initial initsync version 7 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          599762     599762     599762     599762      599762           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:25:1:15::2   0   100       0       0        0    0    0 00:00:00 Idle
2001:25:2:19::2   0   100       0       0        0    0    0 00:00:00 Active
2001:111::151     0   151     734   26620        0    0    0 00:03:17 Active
2001:111::152     0   152  121543   29362   599762    0    0 00:40:06     121498

RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

And we can verify the prefix distribution:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh dpa resources iproute loc 0/7/CPU0

"iproute" OFA Table (Id: 48, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       11           /1       0
 /2       0            /3       0
 /4       11           /5       0
 /6       0            /7       0
 /8       18           /9       13
 /10      40           /11      100
 /12      302          /13      585
 /14      1186         /15      2027
 /16      13440        /17      8190
 /18      13679        /19      24739
 /20      40174        /21      48749
 /22      104394       /23      89388
 /24      489657       /25      91
 /26      0            /27      0
 /28      0            /29      0
 /30      0            /31      0
 /32      99

OFA Infra Stats Summary
                 Create Requests: 3171038
                 Delete Requests: 2334145
                 Update Requests: 8910
                    Get Requests: 0

                  Backwalk Stats
                 Update Requests: 0
                  Update Skipped: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0
                  Retry Attempts: 0
            Recovered from error: 0
               Errors from bwalk: 0

RP/0/RP1/CPU0:5508-1-731#sh dpa resources ip6route loc 0/7/CPU0

"ip6route" OFA Table (Id: 49, Scope: Global)
--------------------------------------------------
IPv6 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       10           /1       0
 /2       0            /3       0
 /4       0            /5       0
 /6       0            /7       0
 /8       0            /9       0
 /10      10           /11      0
 /12      0            /13      0
 /14      0            /15      0
 /16      31           /17      0
 /18      0            /19      1
 /20      13           /21      3
 /22      7            /23      7
 /24      28           /25      8
 /26      15           /27      20
 /28      121          /29      3727
 /30      521          /31      194
 /32      16553        /33      2539
 /34      2226         /35      922
 /36      4777         /37      796
 /38      1460         /39      830
 /40      9269         /41      689
 /42      2760         /43      700
 /44      11231        /45      1077
 /46      2337         /47      2102
 /48      56544        /49      0
 /50      0            /51      0
 /52      1            /53      0
 /54      0            /55      0
 /56      1            /57      0
 /58      0            /59      0
 /60      0            /61      0
 /62      0            /63      0
 /64      30           /65      0
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
 /104     11           /105     0
 /106     0            /107     0
 /108     0            /109     0
 /110     0            /111     0
 /112     0            /113     0
 /114     0            /115     0
 /116     0            /117     0
 /118     0            /119     0
 /120     1            /121     0
 /122     0            /123     1
 /124     1            /125     0
 /126     3            /127     0
 /128     32

OFA Infra Stats Summary
                 Create Requests: 360738
                 Delete Requests: 239129
                 Update Requests: 2
                    Get Requests: 0

                  Backwalk Stats
                 Update Requests: 0
                  Update Skipped: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0
                  Retry Attempts: 0
            Recovered from error: 0
               Errors from bwalk: 0

                          NPU ID: NPU-0                  NPU-1
           Create Server API Err: 0                      0
           Update Server API Err: 0                      0
           Delete Server API Err: 0                      0

RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

### J2 no-eTCAM

Using streaming telemetry, we have a graphical visualization of the counters (having a very small portion of v4/32s in the lab, since they are not announced over the public internet, it only represents a negligeable number and we can trust what is streamed).

![Gauges-J2.png]({{site.baseurl}}/images/Gauges-J2.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh contr npu resources lpm loc 0/7/CPU0

HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 2621440
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 958517   (37 %)
        <mark>iproute                     : 836794   (32 %)</mark>
        <mark>ip6route                    : 121577   (5 %)</mark>
        ipmcroute                   : 101      (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 2621440
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 958517   (37 %)
        iproute                     : 836794   (32 %)
        ip6route                    : 121577   (5 %)
        ipmcroute                   : 101      (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

37% of the capacity with 2021 internet tables, that confirms we don't need a -SE system or card specifically for internet handling. Of course, the -SE options are very relevant for other use-cases, but it's no longer driven by the internet size, at least for a dozen of years.

### J2 with eTCAM

All routes are, as expected, present in eTCAM:

![Gauges-J2-SE.png]({{site.baseurl}}/images/Gauges-J2-SE.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh contr npu resources exttcamipv4 loc 0/3/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv4
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 5000000
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 836868   (17 %)
        <mark>iproute                     : 836893   (17 %)</mark>


RP/0/RP1/CPU0:5508-1-731#sh contr npu resources exttcamipv6

HW Resource Information
    Name                            : ext_tcam_ipv6
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 2000000
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 121559   (6 %)
        <mark>ip6route                    : 121609   (6 %)</mark>

RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

## Projected Internet view (2027) in Jericho2 platforms / LCs

We will reuse the projection done in this article last year to guesstimate the additional routes in year 2027: [https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/](https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/)

![IPv4-projection.png]({{site.baseurl}}/images/IPv4-projection.png){: .align-center}

![IPv6-projection.png]({{site.baseurl}}/images/IPv6-projection.png){: .align-center}

That's 462,640 extra IPv4 and 239,128 extra IPv6 routes.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh bgp sum

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 7531536
BGP main routing table version 7531536
BGP NSR Initial initsync version 7 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         7531536    7531536    7531536    7531536     7531536           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
25.2.19.2         0   100    1016  286333  7069295    0    0 16:53:51          0
192.168.100.151   0  1000  836857  175793  7069295    0    0 01:29:15     836741
192.168.100.152   0  1000  691695     626        0    0    0 01:41:44 Active
192.168.100.153   0  1000    2426  187154        0    0 33564 00:00:09     <mark>462640</mark>
192.168.100.154   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.155   0  1000     753     754        0    0    0 14:25:11 Active
192.168.100.156   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.157   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.158   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.159   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.160   0  1000       0       0        0    0    0 00:00:00 Active
192.168.100.161   0   100       0       0        0    0    0 00:00:00 Active

RP/0/RP1/CPU0:5508-1-731#sh bgp ipv6 un

BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 1317146
BGP main routing table version 1317146
BGP NSR Initial initsync version 7 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         1317146    1317146    1317146    1317146     1317146           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:25:1:15::2   0   100       0       0        0    0    0 00:00:00 Idle
2001:25:2:19::2   0   100       0       0        0    0    0 00:00:00 Active
2001:111::151     0   151    2176   79834  1317146    0    0 00:00:15     <mark>239128</mark>
2001:111::152     0   152  121592   32694  1317146    0    0 01:28:50     121498

RP/0/RP1/CPU0:5508-1-731#sh dpa resources iproute loc 0/7/CPU0

"iproute" OFA Table (Id: 48, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       11           /1       0
 /2       0            /3       0
 /4       11           /5       0
 /6       0            /7       0
 /8       18           /9       13
 /10      40           /11      100
 /12      302          /13      585
 /14      1186         /15      2027
 /16      13440        /17      8190
 /18      13679        /19      24739
 /20      40174        /21      48749
 /22      175966       /23      89388
 /24      875877       /25      91
 /26      0            /27      0
 /28      0            /29      0
 /30      0            /31      0
 /32      100

OFA Infra Stats Summary
                 Create Requests: 4042814
                 Delete Requests: 2748128
                 Update Requests: 21951
                    Get Requests: 0

                  Backwalk Stats
                 Update Requests: 0
                  Update Skipped: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0
                  Retry Attempts: 0
            Recovered from error: 0
               Errors from bwalk: 0

RP/0/RP1/CPU0:5508-1-731#sh dpa resources ip6route loc 0/7/CPU0

"ip6route" OFA Table (Id: 49, Scope: Global)
--------------------------------------------------
IPv6 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       10           /1       0
 /2       0            /3       0
 /4       0            /5       0
 /6       0            /7       0
 /8       0            /9       0
 /10      10           /11      0
 /12      0            /13      0
 /14      0            /15      0
 /16      31           /17      0
 /18      0            /19      1
 /20      13           /21      3
 /22      7            /23      7
 /24      28           /25      8
 /26      15           /27      20
 /28      121          /29      3727
 /30      521          /31      194
 /32      16553        /33      2539
 /34      2226         /35      922
 /36      4777         /37      796
 /38      1460         /39      830
 /40      9269         /41      689
 /42      2760         /43      700
 /44      11231        /45      1077
 /46      2337         /47      2102
 /48      239355       /49      0
 /50      0            /51      0
 /52      1            /53      0
 /54      0            /55      0
 /56      1            /57      0
 /58      0            /59      0
 /60      0            /61      0
 /62      0            /63      0
 /64      56347        /65      0
 /66      0            /67      0
 /68      0            /69      0
 /70      0            /71      0
 /72      0            /73      0
 /74      0            /75      0
 /76      0            /77      0
 /78      0            /79      0
RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

### J2 no-eTCAM

![Gauges-J2-b.png]({{site.baseurl}}/images/Gauges-J2-b.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh contr npu resources lpm loc 0/7/CPU0

HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 2621440
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        <mark>Total In-Use                : 1655437  (63 %)</mark>
        <mark>iproute                     : 1294586  (49 %)</mark>
        <mark>ip6route                    : 360705   (14 %)</mark>
        ipmcroute                   : 101      (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 2621440
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 1655437  (63 %)
        iproute                     : 1294586  (49 %)
        ip6route                    : 360705   (14 %)
        ipmcroute                   : 101      (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

It proves that we will still have a reasonable amount of empty space (a third more, or less) in 2028 if internet growth keeps the current trends.

### J2 with eTCAM

![Gauges-J2-SE-b.png]({{site.baseurl}}/images/Gauges-J2-SE-b.png){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP1/CPU0:5508-1-731#sh contr npu resources exttcamipv4 loc 0/3/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv4
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 5000000
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        <mark>Total In-Use                : 1294661  (26 %)</mark>
        <mark>iproute                     : 1294686  (26 %)</mark>


RP/0/RP1/CPU0:5508-1-731#sh contr npu resources exttcamipv6 loc 0/3/CPU0

HW Resource Information
    Name                            : ext_tcam_ipv6
    Asic Type                       : Jericho 2

NPU-0
OOR Summary
        Estimated Max Entries       : 2000000
        Red Threshold               : 95 %
        Yellow Threshold            : 80 %
        OOR State                   : Green


Current Usage
        Total In-Use                : 360688   (18 %)
        <mark>ip6route                    : 360738   (18 %)</mark>


RP/0/RP1/CPU0:5508-1-731#</code>
</pre>
</div>

Certainly this projection of internet growth is showing we have tons of space in this external TCAM, space available for other data.

## Routes Programming Speed

We performed these tests and documented them for J+ in this article: "NCS5500 FIB Programming Speed" - [https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/](https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/)  
It's now time to do it again for Jericho2.

### Methodology

We are running the test with J2 line cards in modular chassis. It's important to shut down all line card except the one under test since it may impact the performance measurement.  
Indeed line cards will wait for the slowest element of the chassis, that is visible in the following graph with "plateaux" (the moment a line card waits for the lower one to catch up).

![plateaux.png]({{site.baseurl}}/images/plateaux.png){: .align-center}

Also, to make sure we are not "polluting" the test results with a slow announcement of the BGP routes, we are pushing large blocks of routes with an internal Cisco tool. 1M IPv4 prefixes for example.

We will measure the programming speed in LPM in non-eTCAM card and in external TCAM in an -SE card. And the approach will be the same for each test:  
- we will start the advertisement at T0
- at T1, the RIB converged
- at T2, the hardware resource (LPM/eTCAM) is fully programmed
- at T3, we stop the route advertisement (withdrawal begings)
- at T4, all routes are flushed from RIB
- at T5, all routes are flushed from hardware resource (LPM/eTCAM)

### IPv4 in LPM in J2 non-eTCAM

TO: 11:04:18:232, we start advertisement

![1-T0.png]({{site.baseurl}}/images/1-T0.png){: .align-center}

T1: 11:04:27:270 (9 seconds later)

![1-T1.png]({{site.baseurl}}/images/1-T1.png){: .align-center}

First result: RIB is programmed at an average of 1,000,000 / 9 = 111k pfx/s

T2: 11:04:55:276 (37 after beginning of advertisement)

![1-T2.png]({{site.baseurl}}/images/1-T2.png){: .align-center}

Second result: LPM is programmed at an average of 1,000,000 / 37 = 27k pfx/s  
Which is confirmed by this second diagram:

![LPM-prog-1.png]({{site.baseurl}}/images/LPM-prog-1.png){: .align-center}

T3: 11:05:32:282, we stop advertisement

![1-T3.png]({{site.baseurl}}/images/1-T3.png){: .align-center}

T4: 11:05:38:260 (6 seconds later)

![1-T4.png]({{site.baseurl}}/images/1-T4.png){: .align-center}

Third result: RIB is flushed at an average of 1,000,000 / 6 = 166k pfx/s

T5: 11:06:07:248 (35 seconds after beginning of withdrawal)

![1-T5.png]({{site.baseurl}}/images/1-T5.png){: .align-center}

Fourth result: LPM is flushed at an average of 1,000,000 / 35 = 28.5 pfx/s  
Which is also confirmed here:  

![LPM-prog-2.png]({{site.baseurl}}/images/LPM-prog-2.png){: .align-center}

### IPv6 in LPM in J2 non-eTCAM

Let's jump directly to the pfx/s graphs we generated:

![LPMv6-prog-1.png]({{site.baseurl}}/images/LPMv6-prog-1.png){: .align-center}

We program at 30K+ IPv6 pfx/s.

![LPMv6-prog-2.png]({{site.baseurl}}/images/LPMv6-prog-2.png){: .align-center}

And we flush this memory at 28K+ pfx/s.

### IPv4 in eTCAM in J2-SE

Same methodology here, we directly jump to the prefix / second graphs.

![eTCAM-prog-1.png]({{site.baseurl}}/images/eTCAM-prog-1.png){: .align-center}

We program routes in eTCAM at a speed varying between 31K and 32K pfx/s.

![eTCAM-prog-2.png]({{site.baseurl}}/images/eTCAM-prog-2.png){: .align-center}

Flushing is done at a speed between 31K and 35J pfx/s.

### IPv6 in eTCAM in J2-SE

Again, same test methodology for IPv6.

![eTCAMv6-prog-1.png]({{site.baseurl}}/images/eTCAMv6-prog-1.png){: .align-center}

Varies from 25K to 28K pfx/s.

![eTCAMv6-prog-2.png]({{site.baseurl}}/images/eTCAMv6-prog-2.png){: .align-center}

Fluctuates between 24K and 27K pfx/s.

## Conclusion

On Jericho2, the programming and flushing of the routes is more or less the same for both LPM and eTCAM, around 25-30K prefixes per second.  
A system based on Jericho2 with no eTCAM can perfectly handle a full internet view (v4 + v6) and has lot of room for the years to come. eTCAM systems will be used to extend the capacity of the J2 chipset (high interface scale, QoS, etc) via the enablement of specific "-SE" MDB profiles, but the internet size is no longer a criteria to select -SE or non-SE.

## Annex: Telemetry Config

<div class="highlighter-rouge">
<pre class="highlight">
<code>We configured the following sensors on the router for streaming telemetry:

telemetry model-driven
 destination-group DEST-GROUP
  address-family ipv4 192.168.100.142 port 57500
   encoding self-describing-gpb
   protocol grpc no-tls
  !
 !
 sensor-group BGP-COUNTERS
  sensor-path Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv4-unicast/open-config-neighbors/open-config-neighbor/adj-rib-in-post/num-routes/num-routes
  sensor-path Cisco-IOS-XR-ipv4-bgp-oc-oper:oc-bgp/bgp-rib/afi-safi-table/ipv6-unicast/open-config-neighbors/open-config-neighbor/adj-rib-in-post/num-routes/num-routes
 !
 sensor-group FIB-COUNTERS
  sensor-path Cisco-IOS-XR-fib-common-oper:fib/nodes/node/protocols/protocol/vrfs/vrf/summary
 !
 sensor-group OFA-COUNTERS
  sensor-path Cisco-IOS-XR-platforms-ofa-oper:ofa
 !
 sensor-group RIB-COUNTERS
  sensor-path Cisco-IOS-XR-ip-rib-ipv4-oper:rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count
  sensor-path Cisco-IOS-XR-ip-rib-ipv6-oper:ipv6-rib/rib-table-ids/rib-table-id/summary-protos/summary-proto/proto-route-count/active-routes-count
 !
 subscription SUB-GROUP
  sensor-group-id BGP-COUNTERS sample-interval 1000
  sensor-group-id OFA-COUNTERS sample-interval 1000
  sensor-group-id RIB-COUNTERS sample-interval 1000
  destination-id DEST-GROUP
  source-interface MgmtEth0/RP1/CPU0/0
 !
 subscription anx-1622637052563
  sensor-group-id OFA-COUNTERS sample-interval 15000
 !
 subscription anx-1622667923101
  sensor-group-id OFA-COUNTERS sample-interval 15000
 !
 subscription anx-1622714667389
  sensor-group-id OFA-COUNTERS sample-interval 15000
 !
!</code>
</pre>
</div>

**Thanks to:**  
- Fred Cuiller for the corrections

