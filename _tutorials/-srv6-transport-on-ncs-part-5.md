---
published: true
date: '2023-03-28 09:53 +0530'
title: 'SRv6 Transport on NCS5500/5700/500 : Capabilities & Resource Monitoring'
position: hidden
excerpt: SRv6 Platform capabilities and resource monitoring
author: Deepak Balasubramanian
tags:
  - iosxr
---
{% include toc icon="table" title="Table of Contents" %}

|Deepak Balasubramanian, Technical Leader, Technical Marketing (deebalas@cisco.com)|  
|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  


## Overview

This is the 5th in SRv6 series on NCS5500/NCS500 platforms. In this article we will focus on covering the platform capabilties and resources usage when we provision SRv6 services on the NCS5500/5700/500 product-family which is build with BCM DNX Jercho/Jericho+/Jericho2 NPUs.


## Platform Capabilties

Let's get started with a brief overview on capabilties of the platforms under discussion with resepct to SRv6 features & scale. 

Details are also captured in correspondance to different MDB profiles which are supported in NCS5700 platforms. Fore more understanding on MDB pls refer to https://xrdocs.io/ncs5500/tutorials/mdb-ncs5700/


![SRv6-DNX-CAP.png]({{site.baseurl}}/images/SRv6-DNX-CAP.png)

_Table #1 Platform Capabilities_

_Please note: These scale numbers are subjected to change in the future XR releases along with more feature enhancements in the pipeline_

**MDB to PID mapping for reference**


![MDB-PID-MAP.png]({{site.baseurl}}/images/MDB-PID-MAP.png)

_Table #2 Platform MDB mapping_

## SRv6 Manager CLI for details

We have an IOS-XR on-box CLI to check the SRv6 functional and scale capabilities of any the Cisco XR platforms.  CLI : **"show segment-routing srv6 manger"**

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

Every platform with the combination of MDB profiles (for ncs57,) will have the maximum local SID allocation limit (Numbers are shared in the table above) 
We can use the below CLI to check the max SIDs supported on the router,

<div class="highlighter-rouge">
<pre class="highlight">
<code>
sh segment-routing srv6 manager | i Max SID r
  Max SID resources: 12256

</code>
</pre>
</div>

In SRv6, we can support multiple locators for various reasons like slicing/flex-algo. And we enfore max CAP per locator block which can be checked with,

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

We will see in detail about the local SID allocation for different SID functionalities. (lets keep the focus restricted to uSID and not Format1 )

These are some of the important SID functionalities we have today

- uN : Locator SID
- uA : Adjacency SID
- uDT4/6 : Per-VRF SID for GRT/VPN (ipv4/ipv6)
- uDX4/6 : Per-CE SID for GRT/VPN (ipv4/ipv6)
- uDX2 : L2VPN SID for EVPN VPWS
- uB6 : BSID for SRv6 Policy
- uDT2U: EVPN ELAN SID for known-unicast
- uDT2M : EVPN ELAN SID for BUM traffic


Calculaton goes like this,

**MAX SIDs allocated** =  sum (uN:uLocal) + 2*uA

- 16 uN SIDs are reserved by default in hardware to avoid issues with locator configs during OOR conditions
- uLocal here is the local SID on the box which can fall into categories like (uA, uDT4/6, uDX4/6/2 .. etc )
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

**Number of free SID resources:** <mark>4487</mark>  _= ( 12256 - 7769 )_


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
 **OOR**:
    Thresholds (resources): <mark>Green 613</mark>, Warning 368               **/ Global Free resources > 5% of MAX SID for Green/**
    Status: **Resource Available**                                   /*To recover from OOR state, need to be in GREEN threshold*/ 
    **Block fc05:3e::/32**:
        Number of free SID resources: **4946**                       /* Block level MAX free SIDs = 7680 - current allocation from the block*/
        Max SIDs: 7680
        Thresholds: <mark>Green 384</mark>, Warning 231                       **/ > 5% green threshold for Block level max SID/**
        Status: Resource Available
        
</code>
</pre>
</div>     

Global level OOR Threshold set based on SID usage across all the locator blocks (system level)
-Green threshold if (global level free resources > 5% of MAX SID on system level)
-Warning if (global level free resources < 3% of MAX SID on system level)

Block level OOR Threshold set based on SID usage on a specific block
-Green threshold if (Block level free resources > 5% of MAX SID for locator block)
-Warning if (Block level free resources < 3% of MAX SID for locator block)

If we reach the state of OOR “Out of Resources” we will stop programming the SIDs till we go back to GREEN Threshold


## SRv6 Encap resource usage

SRv6 uSIDs(remote) consumes encap resources (in EEDB-Egress Encap DataBase) like the MPLS labels. In NC57 (J2) systems we have encap bank carving in the form of cluster bank pairs which are dedicated to different network applications like SRv6, L3VPN, BGP LU ..etc

SRv6 encap usage can be monitered as srv6nh category which is similar to mplsnh for labels


<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:J2-PE1#sh controllers npu resources encap location 0/4/CPU0 
HW Resource Information
    Name                            : encap
    Asic Type                       : Jericho Two
OFA Table Information
(May not match HW usage)
        ipnh                        : 1038     
        ip6nh                       : 2042     
        mplsnh                      : 79             
       **srv6nh                      : 5185**
       
</code>
</pre>
</div>    

SRv6NH scale depends on the remote SIDs which we use for the H/T.Encap (like encapsulating with DT4/6 or DX4/6/2 SIDs) and T.Insert (which we use for TILFA or SRv6TE scenarios)
We also use the encap resources for SRv6-Policy which is not tied to the H.encap numbers in Table#1

## SRv6 FEC resource usage

FEC table is present in the ingress pipleline of the NPU forwarding block. Basically a prefix/label/SID lookup can point to a FEC entry which will have information about the VOQ of the egress interface and also the encap pointers where the remote labels/SIDs are stored.

In NCS5700 we have 3 levels of FEC hierarchy vs 2 levels in NCS5500/540 systems
Below is an ouput from NCS5700 system where we can see the services SID in H2 FEC and locator SIDs in H3 FEC.

<div class="highlighter-rouge">
<pre class="highlight">
<code>

show controllers npu resources fec location 0/1/CPU0  
    Name: hier_0
           Estimated Max Entries       : 52416   
           Total In-Use                : 8        (0 %)
           OOR State                   : Green
           Bank Info                   : H1 FEC       
       Name: hier_1
           Estimated Max Entries       : 209664  
           Total In-Use                : 45       (0 %)
           OOR State                   : Green
           Bank Info                   : **H2 FEC   >> Services SID**
       Name: hier_2
           Estimated Max Entries       : 78592   
           Total In-Use                : 9    (0 %)
           OOR State                   : Green
           Bank Info                   : **H3 FEC >> uN locator SID**
</code>
</pre>
</div>

 
 ## SRv6 ECMP_FEC resource usage
 
 Similar to FEC we will also use ECMP_FEC for multipath entries pointing to a list of FEC array.
 In NC5700 it maps to similar application hierarchy as that of FEC. 
 This can be monitored with,
 
<div class="highlighter-rouge">
<pre class="highlight">
<code>

show controllers npu resources ecmpfec location 0/RP0/CPU0
Current Hardware Usage
    Name: ecmp_fec
        Estimated Max Entries       : 32768   
        Total In-Use                : 6        (0 %)
        OOR State                   : Green
        Bank Info                   : ECMP 
       Name: hier_0
           Estimated Max Entries       : 30720   
           Total In-Use                : 0        
           OOR State                   : Green
           Bank Info                   : H1 ECMP 
       Name: hier_1
           Estimated Max Entries       : 30720   
           Total In-Use                : 1        
           OOR State                   : Green
           Bank Info                   : H2 ECMP 
       Name: hier_2
           Estimated Max Entries       : 32763   
           Total In-Use                : 5        
           OOR State                   : Green
           Bank Info                   : H3 ECMP

</code>
</pre>
</div>


 ## SRv6 Ultra USID scale with NCS5700
 
 With the encap budget in NCS5700 we can do "Ultra-Scale packing for SRv6 uSIDs"
 
<div class="highlighter-rouge">
<pre class="highlight">
<code>
 <mark>Max-H-Insert    : 4 sids       / 4 Max carriers Inserted/
    Max-H-Encap     : 5 sids    / 5 MAX carriers while encap/</mark>
</code>
</pre>
</div>

Using this encap budget we have succesfully validated 26 usids packing in a single pass. 
(24 Transport uSIDs + 2 Service USIDs)



![Ultras-scale]({{site.baseurl}}/images/Screenshot%202023-03-29%20at%203.25.10%20PM.png)![Screenshot 2023-03-29 at 3.25.10 PM.png]({{site.baseurl}}/images/Screenshot 2023-03-29 at 3.25.10 PM.png)


![Ultra-scale-pcap]({{site.baseurl}}/images/Screenshot 2023-03-29 at 3.25.33 PM.png)



 ##  Show techs required to troublesshoot Srv6 issues
 
<div class="highlighter-rouge">
<pre class="highlight">
<code>  
 
show tech cef
show tech cef platform
show tech l2vpn
show tech l2vpn platform
show tech ofa
show tech segment-routing traffic-eng
show tech routing isis
show tech routing bgp
show tech ipv6 nd  


</code>
</pre>
</div>


## Summary

This article has given an overview on the SRv6 capabilities with respect to the NCS platforms based on the differences in the forwarding asics and the MDB profiles.
We also touched upon the important XR commands and outputs to check on the capabilities, scale of the platform with respect to SRv6 and also covered the resource usage for important on chip databases.

This concludes SRv6 series Part-5. Please stay tuned for more !

 
 
