---
published: false
date: '2022-11-18 09:53 +0530'
title: 'SRv6 Transport on NCS5500/NCS5700/NCS500 : Platform Capabilities & Resources'
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

| Parameters | NCS57 L3MAX SE | NCS7 L3MAX | NCS57 L2MAX SE | NCS57 L2MAX | NCS540 L3MAX | NCS540 L2MAX | NCS55 |
|------------||------------|------------|------------|------------|------------|---------|------|
| Applicable Platforms | NC57 18DD-SE, 36H-SE, 57B1-5DSE, 57C3-MOD-SE |  NC57 24DD, 36H6D-S, MOD-S, 57B1-6D24H, 57C3-MOD-S, 57C1, 57D2 | NC57 18DD-SE, 36H-SE, 57B1-5DSE, 57C3-MOD-SE  | NC57 24DD, 36H6D-S, MOD-S, 57B1-6D24H, 57C3-MOD-S, 57C1, 57D2 | N540-24Q8L2DD-SYS | NC55 Systems |


Number of Locators
16
16
16
16
16
16
8
Max SIDs
12K
6K
16K
11K
3K
6K
8K
SRv6 encap
(H.encap)
16K
16K
12K
16k
10K
12K
-
F1, uSID support
F1, uSID
F1, uSID
F1, uSID
F1, uSID
F1, uSID
F1, uSID
F1, uSID
uDX4/6 in later releases*
uSID: PSP, USD
USP (c8K only)
PSP, USD
PSP, USD
PSP, USD
PSP, USD
PSP, USD
PSP, USD
PSP, USD
SRv6TE
uSID only
781
781
781
781
781
781
Phase1: 732
Phase2: 781
LSS (Large SID stack)
No Recycle
(24 usids)
No Recycle
(24 usids)
No Recycle
(24 usids)
No Recycle
(24 usids)
No Recycle
(24 usids)
No Recycle
(24 usids)
Recycle
(12 usids)
ELAN support 
(uDT2U, uDT2M)
Planned for Future release
Planned for Future release
Planned for Future release
Planned for Future release
Planned for Future release
Planned for Future release
752
MAX SRv6 carriers
H-Insert    : 4 sids
H-Encap   : 5 sids

H-Insert    : 4 sids
H-Encap   : 5 sids
H-Insert    : 4 sids
H-Encap   : 5 sids
H-Insert    : 4 sids
H-Encap   : 5 sids
H-Insert    : 4 sids
H-Encap   : 5 sids
H-Insert    : 4 sids
H-Encap   : 5 sids
H-Insert    : 2 sids
H-Encap   : 2 sids
Overlay Underlay Merge
No
No
No
No
No
No
Yes, with a CLI
L2, L3 propagate differentiation
No
No
No
No
No
No
Yes, 771 with a CLI
