---
published: true
date: '2020-09-30 16:02 +0530'
title: Introduction to NCS55xx and NCS5xx LPTS
author: Tejas Lad
excerpt: Introduction to NCS55xx and NCS5xx LPTS
tags:
  - NCS5500
  - NCS500
  - CoPP
  - control plane
  - LPTS
position: hidden
---
{% include toc %}

## Introduction

![Screenshot 2020-09-30 at 4.49.49 PM.png]({{site.baseurl}}/images/Screenshot 2020-09-30 at 4.49.49 PM.png)

In our previous tech-notes, we focussed on exploring the data-plane security capabilities of NCS5xx and NCS55xx. In this tech-note, we will discuss the NCS55xx and NCS5xx capabilities around control plane protection. In today's networks, control plane security is as important as data plane security. Customers need to focus on protecting the business critical control and management planes from malicious attacks and also misconfiguration events which may overwhelm the processor and bring down the router resulting in an unfortunate event. The Cisco IOS devices have the concept of control plane policing. And in IOS-XR devices, we use a very comprehensive and powerful feature called Local Packet Transport Services. ([Reference](https://community.cisco.com/t5/service-providers-documents/asr9000-xr-local-packet-transport-services-lpts-copp/ta-p/3123792 "Reference")).

In IOS-XR LPTS, as part of **"for-us"** packet delivery process, the rate at which packets are delivered are selectively monitored to avoid overwhelming the CPU. LPTS filters and police the packets based on the defined flow-type rate in hardware before punting to the software. The LPTS  takes the control plane protection (CoPP) concept to a new level by automatically applying rate limiting to all packets that must be handled by any route processor on the device. LPTS is an integral component of IOS-XR systems which provides firewall and policing functionality. LPTS maintains tables describing all packet flows destined for the route processor and uses a set of predefined policers that are applied based on the flow type of the incoming control traffic. The use case of this is we achieve automated control to network health without relying on network operators, as it becomes  difficult to configure these functions manually in large-scale networks.

## LPTS Overview

LPTS performs firewall function by allowing only “for-us” protocol packets into the router for onldropped on the ingress LC itself. LPTS maintains per interface complete table in netio chain in Line card CPU, making sure that packets are delivered to their intended destinations. **IOS XR software** classifies all ‘For Us’ control packets into **97** different flows. Each flow has it own hardware policer to restrict the punt traffic rate for the flow type. In traditional IOS-XR platforms (CRS/ASR9k) LPTS had different regions which consumed 16k entries each. But we do not have the same luxury in NCS55xx and NCS5xx due to limited TCAM resources. We need to use these resources very wisely to accomodate several features and functionalities. Therefore to minimize the usage of resources, the LPTS platform dependent (PD) layer programs only the protocol default entries in hardware and punt the control packets to Line card CPU Netio for full lpts lookup in LPTS Decap node. The Network Input/Output (NetIO) process is responsible for forwarding packets in software. (For deeper understanding of process switching and slow path please [refer](https://www.ciscopress.com/articles/article.asp?p=2272154&seqNum=2#:~:text=Process%20Switching,-Process%20switching%2C%20also&text=In%20IOS%2C%20the%20ip_input%20process,for%20processing%20incoming%20IP%20packets.&text=In%20IOS%20XR%2C%20the%20Network,for%20forwarding%20packets%20in%20software. "refer")). 

LPTS also plays a vital role in support of NSR (Non Stop Routing) enabled control plane. LPTS will make use of the fabric infrastructure for “for-us” control packets to both Active and Standby RP for NSR enabled processes. LPTS also installs dynamic flow entries for certain protocol packets on-demand, an example would be, for ICMP echo request sent out from the local router to peer router, LPTS will create flow entry in Pre-IFIB so that ICMP echo reply received from the peer router will be matched against it.

Note: LPTS is applicable only for control and management plane traffic entering into the router and destined to the local router. Packets originated by the router and transit traffic are not processed by LPTS.
{: .notice--info}


### Main Components of LPTS 

The three component which LPTS needs accomplish the task are: 

- **Port Arbitrator Process**
	- Aggregates bindings into flows
	- Generates IFIB and Pre-IFIB
	- Allocates Flow Group IDs
	- Does not touch packets
- **Flow Manager Process**
	- Copies IFIB slice updates from the Port Arbitrator to netio
	- Does not touch packets
	- Acts as a Sub-Server to the Port Arbitrator
- **Pre-IFIB Manager Process**
	- Copies Pre-IFIB updates from the Port Arbitrator to netio
	- Filters and translates Pre-IFIB updates into TCAM updates
	- Does not touch packets
     

### High Level LPTS Flow Chart (At PI level)

![Screenshot 2020-10-06 at 7.43.53 PM.png]({{site.baseurl}}/images/Screenshot 2020-10-06 at 7.43.53 PM.png)


## LPTS in Pipeline Architecture

![Screenshot 2020-10-05 at 11.36.38 AM.png]({{site.baseurl}}/images/Screenshot 2020-10-05 at 11.36.38 AM.png)

The Programmable Mapping and Filtering (PMF) engine block in IRPP forwarding asic is the most programmable block in the TCAM. It contains all the information of previous blocks like incoming PP port, packet type, forwarding lookup results. It also classifies packets based on predefined protocol fields all the way from Layer 2 to Layer 7. Based on these classifications, various actions can be applied to the packet. In the PMF stage, the packet can be matched and set actions can be applied on the packet. LPTS uses the PMF architecture to classify the packet and other pre-processing attributes to apply action (punt CPU, stats and police).

### NPU and Line Card CPU in action

- Let us the see the packet path in more details. 
- In NPU, the ‘For Us’  control packets undergo hardware TCAM lookup which will hit one of the protocol default TCAM entries. 
- Hardware lookup result contains: 
	- listener_tag (application of interest)
    - flow type 
    - packet destination
    - policer of the flow
    - stats pointer
- All ingress packets are classified into IPv4/IPv6/L2 depending on the L2/L3 headers processed in ingress pipeline.
- After such classification, packets go into Software Path Process (SPP), that selects whether to send to L2/SPIO or NetIO process.
- SPP is a component classifies packets in punt path to identify its client. In our case NetIO or L2/SPIO.
- NetIO handles L3 Protocol Packets like ICMP, OSPF, BGP etc.
- L3 packets undergo a FIB lookup and are classified as a “for-us” packet if it has to be consumed by the router itself.
- Based on lookup result, the control packets will get policed and punted to LC CPU. 
- When the ‘For Us’ packet is received by the LPTS decap node in NetIO, LPTS  does ifib lookup and find the packet associated  protocol client and deliver the packet to the protocol stack.
- This helps us to police all control traffic in hardware while performing full LPTS lookup in software before punting the packet to IOS-XR RP.
- The Streamlined Packet IO - SPIO is used by L2 processes CFM,STP,ARP etc.
- NetIO and SPIO processes the packet header in the received packet to extract packet information and provide it to the client. 
- L2 control protocols do not go through LPTS lookup. They are punted to LC or RP CPU as per configuration. We will look into the details in later sections.
 

**Sample result of a LPTS entry**

We have configured ISIS on the interface Ten0/0/0/6 and Ten0/0/0/7.
Let us check the LPTS hardware entry.

```
RP/0/RP0/CPU0:N55-24#show isis neighbors             
IS-IS 1 neighbors:
System Id      Interface        SNPA           State Holdtime Type IETF-NSF
N540-49        Te0/0/0/7        *PtoP*         Up    28       L2   Capable 
N540-49        Te0/0/0/6        *PtoP*         Up    26       L2   Capable 
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show lpts pifib hardware entry location 0/0/CPU0 | beg TenGigE0/0/0/6
<mark>Interface         : TenGigE0/0/0/6
Is Fragment       : 0
Domain            : 0-default
Listener Tag      : IPv4_STACK
Flow Type         : PIM-mcast-known
DestNode          : Deliver RP0
Dest Type         : Dlvr</mark>
Punt Queue Prio   : MEDIUM
<mark>Hardware NPU Data        
-------      
NPU               : 0
TCAM entry        : 209
Policer_id        : 0x7d78
Stats_id          : 0x800001fd
Stats_hdl         : 0x8ed4ef10
Compression_id    : 0x1
Accepted/Dropped  : 0/0</mark> 
---------------------------------------------------
</code>
</pre>
</div>

As we discussed in the previous sections the hardware lookup results contains various fields like Interface, DestNode, Listener Tag, Flow-type and the hardware NPU data. Similarly we can get the output of default flow types as well. Policing is done on the LC Hardware ASIC before packets hit RP/LC CPU. Each flow policer has default static policer rate value. Each flow policer value can be conifgured from 0 (to drop all packet matching classification criteria) to max of 50K PPS as compared to 100k in ASR9k. Similar to ASR9k platform, the LPTS policers work on a per NPU basis. For example, if the LPTS police value is set to 1000pps for a flow, it means that every NPU on the LC can punt with 1000pps to the CPU for that flow.
 

LPTS takes effect on all applications that receive packets from outside the router. LPTS functions without any need for customer configuration. However, the policer values can be customized if required. You should be very careful while changing this default values.

As we saw the ISIS known default police value is 2100.

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show lpts pifib hardware police location 0/0/CPU0 | in PIM-mcast-known
<mark>PIM-mcast-known        32120   Static  2100      2078      0         0-default</mark>
</code>
</pre>
</div>

We can configure it to a new value and see that it is taking affect. So now if the control packets goes beyond 1000 pps the packets will be policed.

```
RP/0/RP0/CPU0:N55-24(config)#lpts pifib hardware police flow Pim multicast known rate 1000

```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show lpts pifib hardware police location 0/0/CPU0 | in PI$
<mark>PIM-mcast-known        32120   Global  1000      1000      0         0-default</mark>
</code>
</pre>
</div>

**LPTS can use the following tuples to identify a packet:**
    
| Tuples              |
|---------------------|
| VRF                 |
| L3-protocol         |
| L4-protocol         |
| Interface           |
| Source Address      |
| Source Port         |
| Destination Address |
| Destination Port    |


## Supported Flow Types in Hardware LPTS as per IOS-XR 7.2.1

The below command displays pre-Internal Forwarding Information Base (IFIB) information for the designated node.

```
RP/0/RP0/CPU0:N55-24#show lpts pifib hardware police location all 

-------------------------------------------------------------
                Node 0/0/CPU0:
-------------------------------------------------------------
FlowType               Policer Type    Cur. Rate Burst     npu       Domain
---------------------- ------- ------- --------- --------- --------- ---------
Fragment               32102   Static  1000      98        0         0-default
OSPF-mc-known          32103   Static  2000      2000      0         0-default
OSPF-mc-default        32104   Static  100       8         0         0-default
OSPF-uc-known          32105   Static  2000      2000      0         0-default
OSPF-uc-default        32106   Static  100       8         0         0-default
ISIS-known             32107   Static  2000      2000      0         0-default
ISIS-default           32108   Static  100       8         0         0-default
BGP-known              32116   Static  2500      2975      0         0-default
BGP-cfg-peer           32117   Static  2000      2000      0         0-default
BGP-default            32118   Static  100       8         0         0-default
PIM-mcast-default      32119   Static  100       8         0         0-default
PIM-mcast-known        32120   Static  2100      2078      0         0-default
PIM-ucast              32121   Static  1000      500       0         0-default
IGMP                   32122   Static  2000      2000      0         0-default
ICMP-local             32123   Static  2500      250       0         0-default
ICMP-control           32125   Static  2500      250       0         0-default
ICMP-default           32126   Static  1000      98        0         0-default
ICMP-app-default       32127   Static  2500      250       0         0-default
LDP-TCP-known          32128   Static  2500      3104      0         0-default
LDP-TCP-cfg-peer       32129   Static  2000      2000      0         0-default
LDP-TCP-default        32130   Static  100       8         0         0-default
LDP-UDP                32131   Static  2000      2000      0         0-default
All-routers            32132   Static  1000      500       0         0-default
RSVP-default           32138   Static  100       8         0         0-default
RSVP-known             32139   Static  2500      2975      0         0-default
SNMP                   32147   Static  2000      197       0         0-default
SSH-known              32148   Static  1000      98        0         0-default
SSH-default            32149   Static  100       8         0         0-default
HTTP-known             32150   Static  1000      98        0         0-default
HTTP-default           32151   Static  1000      98        0         0-default
SHTTP-known            32152   Static  1000      98        0         0-default
SHTTP-default          32153   Static  1000      98        0         0-default
TELNET-known           32154   Static  1000      98        0         0-default
TELNET-default         32155   Static  100       8         0         0-default
UDP-known              32160   Static  25000     2494      0         0-default
UDP-listen             32161   Static  4000      398       0         0-default
UDP-default            32163   Static  100       8         0         0-default
TCP-known              32164   Static  25000     2494      0         0-default
TCP-listen             32165   Static  25000     2494      0         0-default
TCP-default            32167   Static  100       8         0         0-default
Raw-default            32171   Static  250       23        0         0-default
ip-sla                 32172   Static  1000      98        0         0-default
EIGRP                  32173   Static  1500      1119      0         0-default
PCEP                   32176   Static  100       18        0         0-default
GRE                    32177   Static  150       8         0         0-default
VRRP                   32178   Static  1000      1000      0         0-default
HSRP                   32179   Static  400       158       0         0-default
MPLS-oam               32180   Static  250       31        0         0-default
DNS                    32183   Static  500       49        0         0-default
RADIUS                 32184   Static  7000      4158      0         0-default
TACACS                 32185   Static  500       49        0         0-default
NTP-default            32186   Static  100       8         0         0-default
NTP-known              32187   Static  500       49        0         0-default
DHCPv4                 32193   Static  4000      398       0         0-default
DHCPv6                 32194   Static  4000      398       0         0-default
TPA                    32196   Static  2000      2000      0         0-default
PM-TWAMP               32199   Static  8000      799       0         0-default
```


## Handling Exception and Layer2 Control Packets

Hardware traps are used for handling exception packets (TTLx, Invalid headers), most of Layer2 control protocols (CFM, LACP, BFD, CDP etc) and other system level punt like ACL log, netflow rate, adjaceny, LPTS for-us and prefix miss packets. **RxTrapReceive** is the hardware trap being used to handle for us LPTS punt. All hardware traps are statically programmed with default policer rates per NPU. LPTS module supports configuration of these trap policer values. Same as LPTS punt policers, these trap policers can be configured with policer rate values from 0pps (for complete drop) till predefined max limit per trap. As mentioned above, we need to take care while changing the default values as that will impact both functionality and CPU performance. Like ASR9k, NCS55xx and NCS5xx also processes the L2 frames without LPTS involvement. It has its own policers implemented. Almost each protocol is processed by LC CPU, except bundle control traffic like BFD, OAM. The full list of traps can be checked via the below show command 

```
RP/0/RP0/CPU0:N55-24#show controllers npu stats traps-all instance all location 0/0/CPU0



Trap stats for lc 0


Trap Type                                     NPU  Trap TrapStats   Policer Packet               Packet

                                              ID    ID      ID              Accepted             Dropped
============================================================================================================================
RxTrapMimDiscardMacsaDrop (IRB)               0    1    0x1         32045   0                    0                    
RxTrapMimDiscardMacsaTrap (ERP_BDL)           0    2    0x2         32041   0                    0                    
RxTrapMimDiscardMacsaSnoop(dot1x)             0    3    0x3         32020   0                    0                    
RxTrapMimSaMove(CFM_DOWM_MEP_DMM)             0    6    0x6         32039   0                    0                    
RxTrapMimSaUnknown(RCY_CFM_DOWN_MEP_DMM)      0    7    0x7         32039   0                    0                    
RxTrapAuthSaLookupFail (IPMC default)         0    8    0x8         32035   0                    0                    
RxTrapAuthSaPortFail (L3 wrong MAC)           0    9    0x9         32020   0                    0                    
RxTrapAuthSaVlanFail (L3 unknown-MC/BC)       0    10   0xa         32020   43                   0                    
RxTrapSaMulticast                             0    11   0xb         32020   0                    0                    
RxTrapArpMyIp (Unknown VLAN)                  0    14   0xe         32020   0                    0                    
RxTrapDhcpv4Server                            0    19   0x13        32019   0                    0                    
RxTrapDhcpv4Client                            0    20   0x14        32024   0                    0                    
RxTrapDhcpv6Server                            0    21   0x15        32019   0                    0                    
RxTrapDhcpv6Client                            0    22   0x16        32024   0                    0                    
RxTrapL2Cache_LACP                            0    24   0x18        32003   0                    0                    
RxTrapL2Cache_LLDP1                           0    25   0x19        32004   0                    0                    
RxTrapL2Cache_LLDP2                           0    26   0x1a        32004   0                    0                    
RxTrapL2Cache_ELMI                            0    28   0x1c        32005   0                    0                    
RxTrapL2Cache_BPDU                            0    29   0x1d        32029   0                    0                    
RxTrapL2Cache_BUNDLE_BPDU                     0    30   0x1e        32029   0                    0                    
RxTrapHeaderSizeErr                           0    33   0x21        32020   0                    0                    
RxTrapIpCompMcInvalidIp                       0    37   0x25        32020   0                    0                    
RxTrapMyMacAndIpDisabled                      0    38   0x26        32020   0                    0                    
RxTrapMyMacAndMplsDisable                     0    39   0x27        32020   0                    0                    
RxTrapArpReply                                0    40   0x28        32019   0                    0                    
RxTrapFibDrop                                 0    43   0x2b        32020   0                    0                    
RxTrapMTU                                     0    44   0x2c        32022   0                    0                    
RxTrapMiscDrop                                0    45   0x2d        32020   0                    0                    
RxTrapL2AclDeny                               0    46   0x2e        32036   0                    0                    
Rx_UNKNOWN_PACKET                             0    49   0x31        32020   0                    0                    
RxTrapL3AclDeny                               0    50   0x32        32036   0                    0                    
RxTrapStpStateBlock_EXT_HDR                   0    52   0x34        32048   0                    0                    
RxTrapFailover1Plus1Fail                      0    56   0x38        32043   0                    6                    
RxTrapAdjacentCheckFail (Intf-Down)           0    57   0x39        32043   0                    0                    
RxTrapFcoeSrcIdMismatchSa (EVPN IML drop)     0    58   0x3a        32043   0                    0                    
RxTrapFcoeZoneCheckFail (MAC drop)            0    59   0x3b        32020   0                    0                    
RxTrapOamY1731MplsTp(OAM_SWOFF_DN_CCM)        0    61   0x3d        32031   0                    0                    
RxTrapOamY1731Pwe(OAM_SWOFF_UP_CCM)           0    62   0x3e        32032   0                    0                    
RxTrapOamLevel                                0    69   0x45        32025   0                    0                    
RxTrapRedirectToCpuOamPacket                  0    70   0x46        32027   0                    0                    
RxTrapOamPassive                              0    71   0x47        32026   0                    0                    
RxTrap1588                                    0    72   0x48        32040   0                    0                    
RxTrapExternalLookupError                     0    73   0x49        32020   0                    0                    
RxTrapArplookupFail                           0    74   0x4a        32001   0                    0                    
RxTrapTrillUnknownUc (flooding UC disable)    0    83   0x53        32044   0                    0                    
RxTrapTrillUnknownMc (flooding MC-BC disable) 0    84   0x54        32044   0                    0                    
RxTrapMplsControlWordTrap                     0    88   0x58        32017   0                    0                    
RxTrapMplsControlWordDrop                     0    90   0x5a        32017   0                    0                    
RxTrapMplsUnknownLabel                        0    92   0x5c        32020   0                    0                    
RxTrapIpv4VersionError                        0    100  0x64        32020   0                    0                    
RxTrapIpv4ChecksumError                       0    102  0x66        32020   0                    0                    
RxTrapIpv4HeaderLengthError                   0    104  0x68        32020   0                    0                    
RxTrapIpv4TotalLengthError                    0    106  0x6a        32020   0                    0                    
RxTrapIpv4Ttl0                                0    108  0x6c        32010   0                    0                    
RxTrapIpv4Ttl1                                0    112  0x70        32010   0                    0                    
RxTrapIpv4DipZero                             0    115  0x73        32020   0                    0                    
RxTrapIpv4SipIsMc                             0    117  0x75        32020   0                    0                    
RxTrapIpv6VersionError                        0    120  0x78        32020   0                    0                    
RxTrapIpv6HopCount0                           0    122  0x7a        32013   0                    0                    
RxTrapIpv6HopCount1                           0    123  0x7b        32013   0                    0                    
RxTrapIpv6LoopbackAddress                     0    125  0x7d        32020   0                    0                    
RxTrapIpv6MulticastSource                     0    127  0x7f        32020   0                    0                    
RxTrapIpv6NextHeaderNull                      0    129  0x81        32012   0                    0                    
RxTrapSRv6Cnt3                                0    133  0x85        32020   0                    0                    
RxTrapIpv6Ipv4CompatibleDestination           0    137  0x89        32020   0                    0                    
RxTrapMplsTtl0                                0    141  0x8d        32014   0                    0                    
RxTrapMplsTtl1                                0    142  0x8e        32014   0                    0                    
RxTrapFailoverFacilityInvalid                 0    153  0x99        32020   0                    0                    
RxTrapUcStrictRpfFail                         0    155  0x9b        32037   0                    0                    
bcmRxTrapUcLooseRpfFail                       0    156  0x9c        32037   0                    0                    
RxTrapMcExplicitRpfFail                       0    157  0x9d        32035   0                    0                    
RxTrapOamp(OAM_BDL_DN_NON_CCM)                0    160  0xa0        32033   0                    0                    
RxTrapOamEthUpAccelerated(OAM_BDL_UP_NON_CCM) 0    164  0xa4        32034   0                    0                    
RxTrapReceive                                 0    169  0xa9        32019   4010723              0                    
RxTrapUserDefine_FIB_IPV4_NULL0               0    170  0xaa        32020   0                    0                    
RxTrapUserDefine_SR_Unknown_Label             0    171  0xab        32020   0                    0                    
RxTrapUserDefine_FIB_IPV6_NULL0               0    172  0xac        32020   0                    0                    
RxTrapUserDefine_FIB_IPV4_GLEAN               0    173  0xad        32018   333442               0                    
RxTrapUserDefine_FIB_IPV6_GLEAN               0    174  0xae        32018   0                    0                    
RxTrapUserDefine_IPV4_OPTIONS                 0    175  0xaf        32006   0                    0                    
RxTrapUserDefine_IPV4_RSVP_OPTIONS            0    176  0xb0        32007   0                    0                    
RxTrapUserDefine_OAMP_LBM_LTM                 0    177  0xb1        32028   0                    0                    
RxTrapUserDefine_IPV4_PIM_OPTIONS             0    179  0xb3        32008   0                    0                    
RxTrapUserDefine_RECEIVE_L2                   0    180  0xb4        32019   1114670              0                    
RxNetflowSnoopTrap0                           0    204  0xcc        32020   0                    0                    
RxNetflowSnoopTrap1                           0    205  0xcd        32020   0                    0                    
RxTrapSrv6EndOpPunt                           0    210  0xd2        32047   0                    0                    
bcmRxTrapMacSecurity                          0    211  0xd3        32049   0                    0                    
RxTrapUserDefine_OAM_SAT                      0    214  0xd6        32050   0                    0                    
RxTrapFlowSpecDrop                            0    215  0xd7        32043   0                    0                    
RP/0/RP0/CPU0:N55-24#
```

**Configuring the exception and other protocol packets**

```
RP/0/RP0/CPU0:N55-24(config)#lpts punt police location 0/0/CPU0 ?
  domain     LPTS Domain
  exception  Exception packets(cisco-support)
  protocol   Protocol packets

RP/0/RP0/CPU0:N55-24(config)#lpts punt police location 0/0/CPU0 exception ?
  acl-log               acl log packets(cisco-support)
  adjacency             Glean Adjacency resolution packets(cisco-support)
  fib-drop              pkt drop due no fib match(cisco-support)
  ipv4                  IPV4 exception packets(cisco-support)
  ipv6                  IPv6 packets(cisco-support)
  mpls                  MPLS exception packets(cisco-support)
  netflow               For netflow packets(cisco-support)
  recycle               For recycle packets(cisco-support)
  tunnel-term-fragment  TunnelTermination And Fragmented(cisco-support)
  urpf                  urpf exception packets(cisco-support)

RP/0/RP0/CPU0:N55-24(config)#lpts punt police location 0/0/CPU0 protocol ?
  arp   ARP Packets
  bfd   Bidirectional Forwarding Detection packets(cisco-support)
  cdp   CDP Packets
  cfm   Connectivity Fault Management Protocol packets(cisco-support)
  dhcp  Dynamic Host Configuration Protocol(cisco-support)
  igmp  igmp Protocol packets(cisco-support)
  ipv4  IPv4 packets(cisco-support)
  ipv6  IPv6 packets(cisco-support)
  lacp  LACP Packets
  lldp  LLDP Packets
  mpls  MPLS punt packets(cisco-support)
  pim   Pim Protocol packets(cisco-support)
  ptp   PTP (1588) Protocol packets(cisco-support)
  rsvp  Resource Reservation Protocol packets(cisco-support)

```

### How to check the default policer value of individual traps

```
RP/0/RP0/CPU0:N55-24#show controllers npu stats traps-all instance all locatio$
RxTrapIpv4Ttl0                                0    108  0x6c        32010   0                    0  RxTrapIpv4Ttl1                                0    112  0x70        32010   0                    0 RxTrapMplsTtl0                                0    141  0x8d        32014   0                    0  RxTrapMplsTtl1                                0    142  0x8e        32014   0                    0 
```                  

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#attach location 0/0/CPU0 
Tue Oct  6 16:04:19.751 UTC
Last login: Tue Oct  6 15:56:35 2020 from 172.0.16.1
export PS1='#'
[xr-vm_node0_0_CPU0:~]$export PS1='#'
<mark>#dpa_show puntpolicer | grep -e Def -e 32010
            Def CIR Rate  Conf CIR Rate  CIR Burst         ID
 10 -  0:            100              0        100      32010</mark>
#
</code>
</pre>
</div> 

**Configuring new policer rate**

```
RP/0/RP0/CPU0:N55-24#show running-config lpts 
Tue Oct  6 16:05:05.994 UTC
lpts punt police location 0/0/CPU0
 exception ipv4 ttl-error rate 200
```

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#attach location 0/0/CPU0 
Tue Oct  6 16:05:21.713 UTC
Last login: Tue Oct  6 16:04:19 2020 from 172.0.16.1
export PS1='#'
[xr-vm_node0_0_CPU0:~]$export PS1='#'
#
#dpa_show puntpolicer | grep -e Def -e 32010
            Def CIR Rate  Conf CIR Rate  CIR Burst         ID
 <mark>10 -  0:            100            200        100      32010</mark>
#
</code>
</pre>
</div> 

From the above output we can see that new policer value has been programmed in the hardware.


## Dynamic LPTS Flow Type

In limited hardware resource platforms like NCS55xx and NCS5xx, we cannot support all LPTS flows in the TCAM hardware where resources are shared across multiple features. Therefore there is a need to provide configurable option for customers to decide what flow types they need to program in the hardware and maximum lpts entry per flow type. This can help in saving a lot of hardware resources. Customers can dynamically turn on/off a particular flowtype and max entry for the type using the new CLI to decide the LPTS entries to be programmed in the hardware. This feature is supported from IOS-XR 6.2.x onwards.

### Feature Support

- All the mandatory entries will be programmed in TCAM irrespective of configurations.
- If we do not configure the new CLI, the pifib process will have the default behavior as earlier without any flow limitation or maximum size set.
- For unsupported flow types we will get errors in the platform.
- Flowtype maximum values learned from the configuration will take precedence to the default list.
- Un-configured flows will be set to default static values set by platform.
- Non-Mandatory entries can also be configured using the same CLI.
- The configuration has local scope - meaning we can set different flows and maximum flows per Line Card.
**- Default maximum hardware entries across all lpts flows is 8k.** **(Need to confirm)**

### Flow Type Categories

| Flow Type                            | Details                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|--------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Not supported                        | Flow types not supported by platform. All LPTS PI cli configuration will return error for unsupported flow types.                                                                                                                                                                                                                                                                                                                                                                                                              |
| Supported, Default and Mandatory     | Flow types Supported by platform and not configurable.<br><br>Fragment<br>Raw-Default<br>OSPF Unicast default<br>OSPF Multicast default - 224.0.0.5, 224.0.0.6, ff02::5 and ff02::6<br>BGP default - two entry with src and dest port as 179<br>ICMP-local<br>ICMP-control<br>ICMP-default<br>ICMP-app-default<br>All-routers<br>SSH-default<br>GRE<br>SNMP<br>PIM-mcast-default<br>VRRP<br>PIM-ucast<br>IGMP<br>UDP default<br>TCP default<br>ISIS default<br>RSVP default<br>Telnet default<br>DNS<br>NTP default<br>LDP-UDP |
| Supported, Default and non-mandatory | Default flow types which will be programmed in the hardware at process boot. These flows are also configurable via cli.                                                                                                                                                                                                                                                                                                                                                                                                        |
| Configurable                         | Flows that are configurable via cli (non-default). Download to hardware based on cli config or profile.<br><br><br>BGP-known<br>BGP-cfg-peer<br>LDP-TCP-known<br>LDP-TCP-cfg-peer<br>SSH-known<br>Telnet Known<br>NTP known<br>LDP-UDP<br>OSPF-uc-known<br>OSPF-mc-known<br>RSVP known<br>ISIS known<br>TPA                                                                                                                                                                                                                    |


### Configuring the Dynamic Flows



## Glossary 

| Terms           | Description                                                                                                                                                             |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| For us packets  | Control/management plane packets destined to or to be processed by a node/element in IOS-XR system                                                                      |
| Flow            | A binding between a  tuple (such as protocol, local address, local port, remote address, remote port) and an element                                                    |
| FIB             | Forwarding Information Base – A Table which is used to determine where a packet is to be forwarded.                                                                     |
| iFIB            | Internal Forwarding Information Base – A Table that is used to determine where a “for-us” packet needs to be sent inside IOS-XR running system when pIFIB look up fails |
| LPTS            | Local Packet Transport Services                                                                                                                                         |
| Netio           | An IOS-XR process which performs packet forwarding in software, equivalent of “process switching"                                                                       |
| pFIB or pre-FIB | Compact version of IFIB. A filtered version of pIFIB, HW pIFIB is loaded into LC HW TCAM                                                                                |
| Tuple           | is an ordered list of integer                                                                                                                                           |
| SDR             | Secure Domain Router                                                                                                                                                    |

## Reference

- [ASR9k LPTS](https://community.cisco.com/t5/service-providers-documents/asr9000-xr-local-packet-transport-services-lpts-copp/ta-p/3123792 "ASR9k LPTS")
- [Comparing ASR9k and NCS5500](https://cisco.sharepoint.com/:w:/r/sites/MIGRoutingPMTMECommunications/NCS%205500/Visitor%20Space/PreSales%20Resources/LPTS_Comparision_ASR9k_NCS5500/LPTS%20ASR9000%20and%20NCS5500%20Comparison.docx?d=w65a5b86c044c4e5f9b63a100bdc121f6&csf=1&web=1&e=rXIFkI "Comparing ASR9k and NCS5500")
- [CCO Configuration Guide](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/62x/b-ip-addresses-configuration-guide-ncs5500-62x/b-ipaddr-cg-ncs5500-62x_chapter_0111.html#Cisco_Concept.dita_81657248-82a9-409c-bed1-2109d8552cbf "CCO Configuration Guide")
- [Forwarding Architecture](https://www.ciscolive.com/c/dam/r/ciscolive/us/docs/2014/pdf/BRKSPG%202904.pdf "Forwarding Architecture")
- FRETTA LPTS wiki.

## Summary

Hope this content was useful and you understood the importance of LPTS and its implementation on the platform. To summarize LPTS consists of 3 key functions: 
- Filtering of what can be punted and categorised as a flow.
- Decides where the flow needs to go to: is it LC CPU or RP CPU ?
- Policing of the flows per NPU.
Key message is that, LPTS takes control plane protection to new level and can be considered as a feature which is composed of a dynamic ACL's which are created as part of user configuration automatically. This is configuration driven and no user intervention is required. We also covered how the exception and layer2 control packets are handled in the hardware. In the next tech note we will cover the Dynamic LPTS Flow Type capabilities of the platforms. Stay tuned till then !!!
