---
published: false
date: '2022-11-18 09:53 +0530'
title: 'SRv6 Transport on NCS5500/NCS500 : Capabilities & Monitoring'
position: hidden
excerpt: Resource Consumption with SRv6 transport and overlay services
---
{% include toc icon="table" title="Table of Contents" %}

|Deepak Balasubramanian, Technical Leader, Technical Marketing (deebalas@cisco.com)|  
|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  


## Overview

This will be the fifth article in SRv6 series on NCS5500/NCS500 platforms. In this write-up we will focus on covering the NPU resources usage when we provision SRv6 services on the NCS product family which is build on BCM DNX Jercho/Jericho+/Jericho2 asics.


## Platform Capabilties

Lets get started with a brief overview of the capabilties of the platforms under discussion with resepct to SRv6 features & scale. 

Details are also captured in correspondance to the different MDB profiles we support in NCS5700 platforms. Fore more understanding on MDB pls refer to https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/


![SRv6-DNX-CAP.png]({{site.baseurl}}/images/SRv6-DNX-CAP.png)


MDB to PID mapping for reference


![MDB-PID-MAP.png]({{site.baseurl}}/images/MDB-PID-MAP.png)


## SRv6 Manager CLI for details

We have an IOS-XR on the box CLI to check the functional and scale capabilitie of any Cisco XR platforms.  CLI : **"show segment-routing srb6 manger"**

**Illustration below**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
**“sh segment-routing srv6 manager”**
Parameters:
  SRv6 Enabled: Yes
  SRv6 Operational Mode: 
    Micro-segment:
      SID Base Block: fc05::/24       /*Configured base block*/
  Encapsulation:
    Source Address:
      Configured: ::
      Default: 100:1:1::1                        /*Loopback IP*/
    Hop-Limit: Propagate                   /*Configured Values*/    
    Traffic-class: Propagate               /*Configured Values*/
**Summary**:
  Number of Locators: 15 (15 operational)     /*Current scale*/  
  Number of SIDs: 6749 (0 stale)              /*Current scale*/  
  Max SID resources: 12256           <mark>/Scale Limit J2 L3MAX-SE/</mark>
  Number of free SID resources: 4487
  OOR:
    Thresholds (resources): Green 613, Warning 368 
    Status: <mark>Resource Available</mark>            /*Current OOR state*/
        History: (0 cleared, 0 warnings, 0 full)
        
        Block fc05::/32:
        Number of free SID resources: 7607
        Max SIDs: 7680
        Thresholds: Green 384, Warning 231
        Status: Resource Available
            History: (0 cleared, 0 warnings, 0 full)
_/snipped/_
    Block fc05:4::/32:
        Number of free SID resources: 4946 /*Current Available resources*/
       <mark>Max SIDs: 7680            /Max scale limit per block/</mark>
        Thresholds: Green 384, Warning 231
        Status: Resource Available
            History: (0 cleared, 0 warnings, 0 full)
_/snipped/_
**Platform Capabilities:**
  SRv6: Yes
  TILFA: Yes
  Microloop-Avoidance: Yes
  **Endpoint behaviors:** 
    End.DX6                                  /*F1 per-CE v6*/
    End.DX4                                  /*F1 per-CE v4*/
    End.DT6                                 /*F1 per-VRF v6*/
    End.DT4                                 /*F1 per-VRF v4*/
    End.DX2                                   /*F1 P2P VPWS*/
    End (PSP/USD)                 
    End.X (PSP/USD)
    uN (PSP/USD)                              /*node locator*/
    uA (PSP/USD)                            /*Adjacency usid*/ 
    uDX6                                    /*usid per-CE v6*/
    uDX4                                    /*usid per-CE v4*/
    uDT6                                   /*usid per-VRF v6*/
    uDT4                                   /*usid per-VRF v4*/
    uDX2                                     /*usid P2P VPWS*/
    uB6 (Insert.Red)                       /*BSID for SRv6TE*/
 **Headend behaviors:** 
    T
    H.Insert.Red                        /*SID Insert - LFA,TE*/
    H.Encaps.Red                            /*L3 encap*/                  
    H.Encaps.L2.Red                         /*L2 encap*/

Security rules: 
    SEC-1
    SEC-2
    SEC-3
  Counters: 
    CNT-3
  **Signaled parameters:**
    Max-SL          : 11                                           
    Max-End-Pop-SRH : 5
<mark>Max-H-Insert    : 4 sids       / 4 Max carriers Inserted/
    Max-H-Encap     : 5 sids    / 5 MAX carriers while encap/</mark>
    Max-End-D       : 8
  Configurable parameters (under srv6): 
    Encapsulation: 
      Source Address: Yes
      Hop-Limit     : value=Yes, propagate=Yes
      Traffic-class : value=Yes, propagate=Yes
  Default parameters (under srv6): 
    Encapsulation: 
      Hop-Limit     : value=0, propagate=No
      Traffic-class : value=0, propagate=No
<mark>Max Locators: 16                 /Max scale for locators/
  Max SIDs: 12256                  /Max scale for L3MAX-SE/</mark>
  SID Holdtime: 3 mins             /*Configurable value for SID clean=up*/

</code>
</pre>
</div>




