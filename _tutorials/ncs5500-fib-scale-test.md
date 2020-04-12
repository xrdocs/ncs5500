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

In the video below, we are showing a test with a 36x100G-SE line card used with 6 fabric cards then 5.

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/oglYEDpKsLY?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}




RP/0/RP0/CPU0:OCSE-653#sh bgp sum
Sun Apr 12 15:00:46.176 PDT
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
Sun Apr 12 15:00:54.852 PDT

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

RP/0/RP0/CPU0:OCSE-653#
RP/0/RP0/CPU0:OCSE-653#sh contr npu resources exttcamipv4 loc 0/0/CPU0
Sun Apr 12 14:57:46.554 PDT
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

RP/0/RP0/CPU0:OCSE-653#


