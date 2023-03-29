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

This will be the fifth article in SRv6 series on NCS5500/NCS500 platforms. In this write-up we will focus on covering the platform capabilties for SRv6 and resources usage when we provision SRv6 services on the NCS500/500 product-family which is build with BCM DNX Jercho/Jericho+/Jericho2 asics.


## Platform Capabilties

Lets get started with a brief overview of the capabilties of the platforms under discussion with resepct to SRv6 features & scale. 

Details are also captured in correspondance to the different MDB profiles we support in NCS5700 platforms. Fore more understanding on MDB pls refer to https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/


![SRv6-DNX-CAP.png]({{site.baseurl}}/images/SRv6-DNX-CAP.png)

_Please note: These scale numbers are subjected to change with the future XR releases and more feature enhancements in pipeline as well_

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


## SRv6 MAX SIDs

Every platform with the combination of MDB profiles (for ncs57) will have the maximum local SID allocation limit (Numbers are shared in the table above) 
We can use the below CLI to check the max SIDs,

<div class="highlighter-rouge">
<pre class="highlight">
<code>
sh segment-routing srv6 manager | i Max SID r
  Max SID resources: 12256

</code>
</pre>
</div>

In SRv6 we can support multiple locators for various reasons like slicing/flex-algo. So we enfore max CAP per locator block which can be checked with

<div class="highlighter-rouge">
<pre class="highlight">
<code>
sh segment-routing srv6 manager | i "Block|Max SIDs"
    Block fc05::/32:
        Max SIDs: 7680

</code>
</pre>
</div>

Please note: Sum of the SIDs in different locator blocks should be <= Max SID resources supported 

We will see in details the local SID allocation for different SID functionalities. (lets keep the focus restricted to uSID and not Format1 )

These are some of the mails SID functioanlities we have today

uN : Locator SID
uA : Adjacency SID
uDT4/6 : Per-VRF SID for GRT/VPN (ipv4/ipv6)
uDX4/6 : Per-CE SID for GRT/VPN (ipv4/ipv6)
uDX2 : L2VPN SID for EVPN VPWS
uB6 : BSID for SRv6 Policy
uDT2U: EVPN ELAN SID for known-unicast
uDT2M : EVPN ELAN SID for BUM traffic

Calculaton goes like this,

**MAX SIDs allocated** =  sum (uN:uLocal) + 2*uA

- We now reserve 16 uN SIDs by default in hardware to avoid issues with locator configs during OOR conditions
- uLocal here is the local SID on the box which can fall into categories like (uA, uDT4/6, uDX4/6/2)
- uA is counted 3 times, once as a part of uN:uLocal (uLocal = uA),and twice for local-only uA

**Example:**

<div class="highlighter-rouge">
<pre class="highlight">
<code>

sh segment-routing srv6 manager 
Summary:
  Number of Locators: 15 (15 operational)                            /*uN or Locators*/
  Number of SIDs: <mark>6749</mark> (0 stale)                                     /* SID usage*/
  Max SID resources: <mark>12256</mark>
  Number of free SID resources: <mark>4487 </mark>                                /* 12256 - 6749 != 4487 :) */
  OOR:
    Thresholds (resources): Green 613, Warning 368                   /* Free resources > 5% of MAX SID for Green*/
    Status: Resource Available                                /*To recover from OOR state, need to be in GREEN threshold*/ 

sh segment-routing srv6 sid | i uA | ut wc
    <mark>510</mark>    4080   56100
RP/0/RP0/CPU0:J2-PE1#sh segment-routing srv6 sid | i "uDT|uDX|uB" | ut wc
  <mark> 6239/</mark>   37482  686534
  
</code>
</pre>
</div>

**Number of SIDs:** **6749** _= sum of uLocals (uA|uDT|uDX|uB)_ 

**Actual SID resource usage**: _6749 + 2*510 (uA) =_ **7769**

**Number of free SID resources:** **4487**  _= ( 12256 - 7769 )_


## OOR (Out Of Resource) monitoring for MAX SIDs

**Illustration:**

<div class="highlighter-rouge">
<pre class="highlight">
<code>
sh segment-routing srv6 manager 
Summary:
  Number of Locators: 15 (15 operational)                         
  Number of SIDs: **6749** (0 stale)                                  
  Max SID resources: **12256**
  Number of free SID resources: **4487**                             
 OOR:
    Thresholds (resources): <mark>Green 613</mark>, Warning 368               **/ Global Free resources > 5% of MAX SID for Green/**
    Status: **Resource Available**                                   /*To recover from OOR state, need to be in GREEN threshold*/ 
    Block fc05:3e::/32:
        Number of free SID resources: **4946**                       /* Block level MAX free SIDs = 7680 - current allocation from the block*/
        Max SIDs: 7680
        Thresholds: <mark>Green 384</mark>, Warning 231                       **/ > 5% green threshold for Block level max SID/**
        Status: Resource Available
        
</code>
</pre>
</div>     

Global level SID allocation and usage is the MAX across all the locator blocks.
 If (Global OOR thresholds and Free entries > 5 % of MAX SID will be Green
- If we reach the status “Out of Resources” we will stop programming the SIDs till we go back to GREEN Threshold
- We also CAP each locator block at 7K and the Green threshold is 5% of 7k and the same rules apply
- In latest release we pre-carve uN resources for 16 locators in hardware so we can handle new locator provisioning even during OOR state .





 