---
published: true
date: '2019-12-12 16:37 +0100'
title: Label Stack Imposition Enhancement on NCS5500
author: Jose Liste
excerpt: >-
  Presentation and demonstration of the label stack imposition enhancements
  brought with IOS XR 7.0.1 on NCS5500
position: hidden
---
{% include toc icon="table" title="Label Stack Imposition Enhancement on NCS5500" %}

You can find more content related to NCS5500 including routing memory management, URPF, ACLs, Netflow following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

INTRO

## Demo/Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/nkL301rckL4?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## Default ASIC capabilities


![LLS-ncs5500-before-transport-fig.png]({{site.baseurl}}/images/LLS-ncs5500-before-transport-fig.png){: .align-center}

![LLS-ncs5500-before-trans_service-fig.png]({{site.baseurl}}/images/LLS-ncs5500-before-trans_service-fig.png){: .align-center}


Shortcut in the video123: [https://youtu.be/nkL301rckL4?t=75](https://youtu.be/nkL301rckL4?t=75)

## Details on the enhancement


![LLS-ncs5500-after-transport-fig.png]({{site.baseurl}}/images/LLS-ncs5500-after-transport-fig.png){: .align-center}

![LLS-ncs5500-after-trans_service-fig.png]({{site.baseurl}}/images/LLS-ncs5500-after-trans_service-fig.png){: .align-center}

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=229](https://youtu.be/nkL301rckL4?t=229)

## Demo

### Test topology

![LLS-ncs5500-topo-components-fig.png]({{site.baseurl}}/images/LLS-ncs5500-topo-components-fig.png){: .align-center}

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=428](https://youtu.be/nkL301rckL4?t=428)

### Service deployment with NSO

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=707](https://youtu.be/nkL301rckL4?t=707)

### SR On-Demand Next Hop (ODN)

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1030](https://youtu.be/nkL301rckL4?t=1030)

### Monitoring with Cisco Crossworks Optimization Engine

![LLS-ncs5500-coe-topo-fig.png]({{site.baseurl}}/images/LLS-ncs5500-coe-topo-fig.png){: .align-center}

![LLS-ncs5500-coe-disj-1-fig.png]({{site.baseurl}}/images/LLS-ncs5500-coe-disj-1-fig.png){: .align-center}

![LLS-ncs5500-coe-disj-2-fig.png]({{site.baseurl}}/images/LLS-ncs5500-coe-disj-2-fig.png){: .align-center}

![LLS-ncs5500-coe-disj-1-details-fig.png]({{site.baseurl}}/images/LLS-ncs5500-coe-disj-1-details-fig.png){: .align-center}


Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1372](https://youtu.be/nkL301rckL4?t=1372)

### Link failure before enhancement

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1551](https://youtu.be/nkL301rckL4?t=1551)


### Test with the improvement

```
RP/0/0/CPU0:XTC-7#sh run pce
Mon Nov 18 01:04:24.646 UTC
pce
 address ipv4 1.1.1.207
 rest
 !
 disjoint-path
  group-id 111 type node
   lsp 1 pcc ipv4 1.1.1.6 lsp-name fooIGPDISJnode
   lsp 2 pcc ipv4 1.1.1.5 lsp-name barIGPDISJnode
  !
  group-id 222 type link
   lsp 1 pcc ipv4 1.1.1.6 lsp-name fooIGPDISJlink
   lsp 2 pcc ipv4 1.1.1.5 lsp-name barIGPDISJlink
  !
  group-id 775 type srlg
   lsp 1 pcc ipv4 1.1.1.4 lsp-name bgp_c_11000_ep_1.1.1.6_discr_100 shortest-path
   lsp 2 pcc ipv4 1.1.1.2 lsp-name bgp_c_11000_ep_1.1.1.10_discr_100
  !
  group-id 776 type srlg
   lsp 1 pcc ipv4 1.1.1.6 lsp-name bgp_c_10000_ep_1.1.1.4_discr_100 shortest-path
   lsp 2 pcc ipv4 1.1.1.10 lsp-name bgp_c_10000_ep_1.1.1.2_discr_100
  !
 !
 logging
  no-path
  fallback
 !
 password encrypted 060506324F41
 backoff
  threshold 20
 !
!
```
```
RP/0/0/CPU0:XTC-7#show pce association group-id 776
Mon Nov 18 01:04:48.236 UTC

PCE's association database:
----------------------
Association: Type SRLG-Disjoint, Group 776, Not Strict
 Associated LSPs:
  LSP[0]:
   PCC 1.1.1.6, tunnel name bgp_c_10000_ep_1.1.1.4_discr_100,  PLSP ID 53, tunnel ID 35, LSP ID 1, Configured on PCE
  LSP[1]:
   PCC 1.1.1.10, tunnel name bgp_c_10000_ep_1.1.1.2_discr_100,  PLSP ID 20, tunnel ID 11, LSP ID 2, Configured on PCE
 Status: Satisfied

RP/0/0/CPU0:XTC-7#show pce lsp name bgp_c_10000_ep_1.1.1.2_discr_100
Mon Nov 18 01:05:25.335 UTC

PCE's tunnel database:
----------------------
PCC 1.1.1.10:

Tunnel Name: bgp_c_10000_ep_1.1.1.2_discr_100
 LSPs:
  LSP[0]:
   source 1.1.1.10, destination 1.1.1.2, tunnel ID 11, LSP ID 2
   State: Admin up, Operation up
   Setup type: Segment Routing
   Binding SID: 80010
   Maximum SID Depth: 5
   Absolute Metric Margin: 0
   Relative Metric Margin: 0%
```
```
RP/0/0/CPU0:XTC-7#show pce lsp name bgp_c_10000_ep_1.1.1.2_discr_100 detail
Mon Nov 18 01:05:29.935 UTC

PCE's tunnel database:
----------------------
PCC 1.1.1.10:

Tunnel Name: bgp_c_10000_ep_1.1.1.2_discr_100
 LSPs:
  LSP[0]:
   source 1.1.1.10, destination 1.1.1.2, tunnel ID 11, LSP ID 2
   State: Admin up, Operation up
   Setup type: Segment Routing
   Binding SID: 80010
   Maximum SID Depth: 5
   Absolute Metric Margin: 0
   Relative Metric Margin: 0%
   Preference: 100
   Bandwidth: signaled 0 kbps, applied 0 kbps
   PCEP information:
     PLSP-ID 0x14, flags: D:1 S:0 R:0 A:1 O:1 C:0
   LSP Role: Disjoint LSP
   State-sync PCE: None
   PCC: 1.1.1.10
   LSP is subdelegated to: None
   Reported path:
     Metric type: IGP, Accumulated Metric 130
      SID[0]: Adj, Label 80011, Address: local 11.8.10.10 remote 11.8.10.8
      SID[1]: Node, Label 16007, Address 1.1.1.7
      SID[2]: Node, Label 16001, Address 1.1.1.1
      SID[3]: Adj, Label 80005, Address: local 10.1.2.1 remote 10.1.2.2
   Computed path: (Local PCE)
     Computed Time: Mon Nov 18 01:03:20 UTC 2019 (00:02:10 ago)
     Metric type: IGP, Accumulated Metric 130
      SID[0]: Adj, Label 80011, Address: local 11.8.10.10 remote 11.8.10.8
      SID[1]: Node, Label 16007, Address 1.1.1.7
      SID[2]: Node, Label 16001, Address 1.1.1.1
      SID[3]: Adj, Label 80005, Address: local 10.1.2.1 remote 10.1.2.2
   Recorded path:
     None
   Disjoint Group Information:
     Type SRLG-Disjoint, Group 776
```
```
RP/0/RP0/CPU0:Node-10#show segment-routing traffic-eng policy color 10000
Sun Nov 17 16:03:59.996 PST

SR-TE policy database
---------------------

Color: 10000, End-point: 1.1.1.2
  Name: srte_c_10000_ep_1.1.1.2
  Status:
    Admin: up  Operational: up for 3d11h (since Nov 14 04:58:01.022)
  Candidate-paths:
    Preference: 200 (BGP ODN) (shutdown)
      Requested BSID: dynamic
      PCC info:
        Symbolic name: bgp_c_10000_ep_1.1.1.2_discr_200
        PLSP-ID: 21
        Maximum SID Depth: 5
      Dynamic (invalid)
        Metric Type: IGP,   Path Accumulated Metric: 0
    Preference: 100 (BGP ODN) (active)
      Requested BSID: dynamic
      PCC info:
        Symbolic name: bgp_c_10000_ep_1.1.1.2_discr_100
        PLSP-ID: 20
        Maximum SID Depth: 5
      Dynamic (pce 1.1.1.207) (valid)
        Metric Type: IGP,   Path Accumulated Metric: 130
          80011 [Adjacency-SID, 11.8.10.10 - 11.8.10.8]
          16007 [Prefix-SID, 1.1.1.7]
          16001 [Prefix-SID, 1.1.1.1]
          80005 [Adjacency-SID, 10.1.2.1 - 10.1.2.2]
  Attributes:
    Binding SID: 80010
    Forward Class: 0
    Steering labeled-services disabled: no
    Steering BGP disabled: no
    IPv6 caps enable: yes
```
```
RP/0/RP0/CPU0:Node-10#show isis adjacency gigabitEthernet 0/0/0/2
Sun Nov 17 16:15:11.803 PST

IS-IS 1 Level-1 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD

IS-IS 1 Level-2 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD
Node-8         Gi0/0/0/2                *PtoP*         Up    24   3d11h    Yes None None

Total adjacency count: 1
```
```
RP/0/RP0/CPU0:Node-10#show isis adjacency gigabitEthernet 0/0/0/2 detail
Sun Nov 17 16:15:17.418 PST

IS-IS 1 Level-1 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD

IS-IS 1 Level-2 adjacencies:
System Id      Interface                SNPA           State Hold Changed  NSF IPv4 IPv6
                                                                               BFD  BFD
Node-8         Gi0/0/0/2                *PtoP*         Up    28   3d11h    Yes None None
  Area Address:           49.0000
  Neighbor IPv4 Address:  11.8.10.8*
  Adjacency SID:          80011 (protected)
   Backup label stack:    [16008]
   Backup stack size:     1
   Backup interface:      Te0/0/0/3
   Backup nexthop:        10.6.10.6
   Backup node address:   1.1.1.8
  Non-FRR Adjacency SID:  80012
  Neighbor IPv6 Address:  fe80::aa0c:dff:fe00:89bb*
  Topology:               IPv4 Unicast (parallel adjacency suppressed)
  Topology:               IPv6 Unicast (parallel adjacency suppressed)
  BFD Status:             BFD Not Required, Neighbor Useable

Total adjacency count: 1
```
```
RP/0/RP0/CPU0:Node-10#show mpls forwarding labels 80011 detail
Sun Nov 17 16:14:51.170 PST
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
80011  Pop         SR Adj (idx 1)     Gi0/0/0/2    11.8.10.8       0
     Updated: Nov 14 05:00:48.474
     Path Flags: 0x400 [  BKUP-IDX:1 (0x993f3cf0) ]
     Version: 7292, Priority: 1
     Label Stack (Top -> Bottom): { Imp-Null }
     NHID: 0x0, Encap-ID: 0x1383000000002, Path idx: 0, Backup path idx: 1, Weight: 10
     MAC/Encaps: 14/14, MTU: 1500
     Outgoing Interface: GigabitEthernet0/0/0/2 (ifhandle 0x00000180)
     Packets Switched: 0

       16008       SR Adj (idx 1)     Te0/0/0/3    10.6.10.6       0            (!)
     Updated: Nov 14 05:00:48.474
     Path Flags: 0x100 [  BKUP, NoFwd ]
     Version: 7292, Priority: 1
     Label Stack (Top -> Bottom): { 16008 }
     NHID: 0x0, Encap-ID: 0x1386600000002, Path idx: 1, Backup path idx: 0, Weight: 10
     MAC/Encaps: 14/18, MTU: 1500
     Outgoing Interface: TenGigE0/0/0/3 (ifhandle 0x00000190)
     Packets Switched: 0
     (!): FRR pure backup
```
```
RP/0/RP0/CPU0:Node-10#show segment-routing traffic-eng forwarding policy color 10000
Sun Nov 17 16:04:04.683 PST

SR-TE Policy Forwarding database
--------------------------------

Color: 10000, End-point: 1.1.1.2
  Name: srte_c_10000_ep_1.1.1.2
  Binding SID: 80010
  Active LSP:
    Candidate path:
      Preference: 100 (BGP)
    Local label: 80013
    Segment lists:
      SL[0]:
        Name: dynamic
        Packets/Bytes Switched: 298994357/302582289284
        Paths:
          Path[0]:
            Outgoing Label: 16007
            Outgoing Interface: GigabitEthernet0/0/0/2
            Next Hop: 11.8.10.8
            Switched Packets/Bytes: 298994357/302582289284
            FRR Pure Backup: No
            ECMP/LFA Backup: No
            Label Stack (Top -> Bottom): { 16007, 16001, 80005 }
          Path[1]:
            Outgoing Label: 16008
            Outgoing Interface: TenGigE0/0/0/3
            Next Hop: 10.6.10.6
            Switched Packets/Bytes: 0/0
            FRR Pure Backup: Yes
            ECMP/LFA Backup: No
            Label Stack (Top -> Bottom): { 16008, 16007, 16001, 80005 }

  Policy Packets/Bytes Switched: 299131510/302720539484
```
```
RP/0/RP0/CPU0:Node-10#show mpls forwarding labels 80013 detail
Sun Nov 17 16:05:52.195 PST
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
80013  16007       SR TE: 11 [TE-INT]  Gi0/0/0/2    11.8.10.8       302691179472
     Updated: Nov 14 05:00:49.908
     Path Flags: 0x400 [  BKUP-IDX:1 (0x993f4850) ]
     Version: 7310, Priority: 2
     Label Stack (Top -> Bottom): { 16007 16001 80005 }
     NHID: 0x0, Encap-ID: 0x1382900000001, Path idx: 0, Backup path idx: 1, Weight: 64
     MAC/Encaps: 14/26, MTU: 1500
     Outgoing Interface: GigabitEthernet0/0/0/2 (ifhandle 0x00000180)
     Packets Switched: 299101956

       80014       SR TE: 11 [TE-INT]  Te0/0/0/3    10.6.10.6       0            (!)
     Updated: Nov 14 05:00:49.908
     Path Flags: 0x100 [  BKUP, NoFwd ]
     Version: 7310, Priority: 2
     Label Stack (Top -> Bottom): { 80014 16001 80005 }
     NHID: 0x0, Encap-ID: 0x1388600000001, Path idx: 1, Backup path idx: 0, Weight: 64
     MAC/Encaps: 14/26, MTU: 1500
     Outgoing Interface: TenGigE0/0/0/3 (ifhandle 0x00000190)
     Packets Switched: 0
     (!): FRR pure backup
```
```
RP/0/RP0/CPU0:Node-10#show mpls forwarding labels 80013 private detail hardware ingress location 0/0/CPU0
Sun Nov 17 16:10:08.889 PST
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
80013  16007       SR TE: 11 [TE-INT]  Gi0/0/0/2    11.8.10.8       N/A
     Updated: Nov 14 05:00:49.908
     Path Flags: 0x400 [  BKUP-IDX:1 (0x8b092dc0) ]
     Version: 7310, Priority: 2
     Label Stack (Top -> Bottom): { 16007 16001 80005 }
     NHID: 0x8, Encap-ID: 0x1382900000001, Path idx: 0, Backup path idx: 1, Weight: 64
     MAC/Encaps: 14/26, MTU: 8986
     Outgoing Interface: GigabitEthernet0/0/0/2 (ifhandle 0x00000180)
     Packets Switched: 0

       80014       SR TE: 11 [TE-INT]  Te0/0/0/3    10.6.10.6       N/A          (!)
     Updated: Nov 14 05:00:49.908
     Path Flags: 0x100 [  BKUP, NoFwd ]
     Version: 7310, Priority: 2
     Label Stack (Top -> Bottom): { 80014 16001 80005 }
     NHID: 0x6, Encap-ID: 0x1388600000001, Path idx: 1, Backup path idx: 0, Weight: 64
     MAC/Encaps: 14/26, MTU: 8986
     Outgoing Interface: TenGigE0/0/0/3 (ifhandle 0x00000190)
     Packets Switched: 0
     (!): FRR pure backup



 LEAF - HAL pd context :
 sub-type : MPLS, ecd_marked:0, has_collapsed_ldi:0
 collapse_bwalk_required:0, ecdv2_marked:0,
HW Walk:
LEAF:
    PI:0x308cd24ab8 PD:0x308cd24b58 rev:221762 type: MPLS (2)
    LEAF location: LEM
    FEC key: 0

    LWLDI:
        PI:0x308b6ea120 PD:0x308b6ea160 rev:221761 p-rev:221760  ldi type:EOS0_EOS1
        FEC key: 0x1d5040000f8f fec index: 0x0(0) num paths:1, bkup paths: 1
        IMP LDI:
        IMP pattern:3
        PI:0x308b6ea120 PD:0x308b6ea160 rev:221761 p-rev:221760
        FEC key: 0x1d5140000f8f fec index: 0x2001ff64(130916) num paths:1
        Path:0  fec index: 0x2001ff64(130916) DSP:0x20
                MPLS encap key: 0xf1b0000040013829 MPLS encap id: 0x40013829 Remote: 0
                MPLS Backup Recycle label: 80014
        Path:1  fec index: 0x2001ff65(130917) Flow:0xa0
                MPLS encap key: 0xf1b0000040013886 MPLS encap id: 0x40013886 Remote: 0

        SHLDI:
            PI:0x308f181550 PD:0x308f181620 rev:221760 dpa-rev:4730878 flag:0x0
            FEC key: 0x1d5040000f8f fec index: 0x2001ffa0(130976) num paths: 1 bkup paths: 1
            p-rev:221230 221480
            Path:0 fec index: 0x2001ffa0(130976) DSP:0x20 Dest fec index: 0x0(0)
            Path:1 fec index: 0x2001ffa1(130977) DSP:0x1f Dest fec index: 0x0(0)
                  BKUP-FRRNH:
                  PI:0x308b092dc0 PD:0x308b092e90 Rev: 221480, p-rev: 208093 221479 , Flag:0x0
                  PROT-FRRNH:
                  PI:0x308b088bc0 PD:0x308b088c90 Rev: 221479 Prot_eng dpa-rev: 4730679 dpa-rev: 4730679
                  FRR Active: 0, FRR Protect Handle: 0x308ce8c100, Id 0x4000005c(0x4000005c), Status 1
```
```
RP/0/RP0/CPU0:Node-10#show mpls forwarding labels 80014 detail
Sun Nov 17 16:06:08.456 PST
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes
Label  Label       or ID              Interface                    Switched
------ ----------- ------------------ ------------ --------------- ------------
80014  16008       No ID              Te0/0/0/3    10.6.10.6       0
     Updated: Nov 14 05:00:49.899
     Version: 7306, Priority: 1
     Label Stack (Top -> Bottom): { 16008 16007 }
     NHID: 0x0, Encap-ID: 0x1389c00000003, Path idx: 0, Backup path idx: 0, Weight: 0
     MAC/Encaps: 14/22, MTU: 1500
     Outgoing Interface: TenGigE0/0/0/3 (ifhandle 0x00000190)
     Packets Switched: 0
```
```
RP/0/RP0/CPU0:Node-10#show install active
Sun Nov 17 16:03:53.156 PST
Node 0/RP0/CPU0 [RP]
  Boot Partition: xr_lv97
  Active Packages: 9
        ncs5500-xr-7.0.1 version=7.0.1 [Boot image]
        ncs5500-mpls-te-rsvp-3.1.0.0-r701
        ncs5500-isis-2.1.0.0-r701
        ncs5500-mpls-2.1.0.0-r701
        ncs5500-ospf-2.0.0.0-r701
        ncs5500-li-1.0.0.0-r701
        ncs5500-mgbl-3.0.0.0-r701
        ncs5500-mcast-3.0.0.0-r701
        ncs5500-k9sec-3.1.0.0-r701

Node 0/0/CPU0 [LC]
  Boot Partition: xr_lcp_lv98
  Active Packages: 9
        ncs5500-xr-7.0.1 version=7.0.1 [Boot image]
        ncs5500-mpls-te-rsvp-3.1.0.0-r701
        ncs5500-isis-2.1.0.0-r701
        ncs5500-mpls-2.1.0.0-r701
        ncs5500-ospf-2.0.0.0-r701
        ncs5500-li-1.0.0.0-r701
        ncs5500-mgbl-3.0.0.0-r701
        ncs5500-mcast-3.0.0.0-r701
        ncs5500-k9sec-3.1.0.0-r701
```
 
Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1936](https://youtu.be/nkL301rckL4?t=1936)

### Service removal with NSO

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=2178](https://youtu.be/nkL301rckL4?t=2178)

## Recirculation impact


## Conclusion






## to delete / templates / examples

<div class="highlighter-rouge">
<pre class="highlight">
<code>CONFIG / SHOW COMMANDS / <mark>HIGHLIGHT</mark> / ETC EXAMPLE</code>
</pre>
</div>

Note to highlight: 

**Update**: blablabla
{: .notice--info}

Aligned image:  
![core-0-1-.png]({{site.baseurl}}/images/core-0-1-.png){: .align-center}
