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

In our previous tech-notes, we focussed on exploring the data-plane security capabilities of NCS5xx and NCS55xx. In continuation to the same, we will discuss the NCS55xx and NCS5xx capabilities around control plane protection. In today's networks, control plane security is as important as data plane security. Customers need to focus on protecting the business critical control and management planes from malicious attacks and also misconfiguration events which may overwhelm the processor and bring down the router resulting in an unfortunate event. The Cisco IOS devices have the concept of control plane policing. In IOS-XR devices, we use a very comprehensive and powerful feature called Local Packet Transport Services. ([Reference](https://community.cisco.com/t5/service-providers-documents/asr9000-xr-local-packet-transport-services-lpts-copp/ta-p/3123792 "Reference")) for control plane protection.

In IOS-XR LPTS, as part of **for-us** packet delivery process, the rate at which packets are delivered are selectively monitored to avoid overwhelming the CPU. LPTS filters and police the packets based on the defined flow-type rate in hardware before punting to the software. The LPTS  takes the control plane protection (CoPP) concept to a new level by automatically applying rate limiting to all packets that must be handled by any route processor on the device. LPTS is an integral component of IOS-XR systems which provides firewall and policing functionality. LPTS maintains tables describing all packet flows destined for the route processor and uses a set of predefined policers that are applied based on the flow type of the incoming control traffic. The use case of this is we achieve automated control to network health without relying on network operators as it becomes  difficult to configure these functions manually in large-scale networks.

## LPTS Overview: NCS55xx and NCS5xx

LPTS module in **IOS XR software** classifies all ‘For Us’ control packets into **97** different flows. Each flow has it own hardware policer to restrict the punt traffic rate for the flow type. In traditional IOS-XR platforms (CRS/ASR9k) LPTS had different regions which consumed 16k entries each. But we do not have the same luxury in NCS55xx and NCS5xx due to limited TCAM resources. We need to use the TCAM very wisely to accomodate several features and functionalities. Therefore to minimize the usage of TCAM resources,  the LPTS platform dependent layer programs only the protocol default entries in hardware and punt the control packets to Line card CPU Netio for full lpts lookup in LPTS Decap node. In IOS XR, the Network Input/Output (NetIO) process is the equivalent to the IOS ip_input process and is responsible for forwarding packets in software. (For deeper understanding of process switching and slow path please [refer](https://www.ciscopress.com/articles/article.asp?p=2272154&seqNum=2#:~:text=Process%20Switching,-Process%20switching%2C%20also&text=In%20IOS%2C%20the%20ip_input%20process,for%20processing%20incoming%20IP%20packets.&text=In%20IOS%20XR%2C%20the%20Network,for%20forwarding%20packets%20in%20software. "refer")). LPTS also maintains per interface complete table in netio chain in Line card CPU,  making sure that packets are delivered to their intended destinations. LPTS uses two components to accomplish this task: the port arbitrator and flow managers. The port arbitrator and flow managers are processes that maintain the tables that describe packet flows for a logical router, known as the Internal Forwarding Information Base (IFIB). The IFIB is used to route received packets to the correct Route Processor for processing. ([reference](https://www.cisco.com/c/en/us/td/docs/iosxr/ncs5500/ip-addresses/62x/b-ip-addresses-configuration-guide-ncs5500-62x/b-ipaddr-cg-ncs5500-62x_chapter_0111.html#Cisco_Concept.dita_365eb914-2cee-4691-98ec-9c3a73fd6c4c "reference")). LPTS also plays a vital role in support of NSR (Non Stop Routing) enabled control plane. LPTS will make use of the fabric infrastructure the “for-us” control packets to both Active and Standby RP for NSR enabled processes.


## LPTS in Pipeline Architecture

![Screenshot 2020-10-05 at 11.36.38 AM.png]({{site.baseurl}}/images/Screenshot 2020-10-05 at 11.36.38 AM.png)

The Programmable Mapping and Filtering (PMF) engine block in IRPP forwarding asic is the most programmable block in the TCAM. It contains all the information of previous blocks like incoming PP port, packet type, forwarding lookup results. It also classifies packets based on predefined protocol fields all the way from Layer 2 to Layer 7. Based on these classifications, various actions can be applied to the packet. In the PMF stage, the packet can be matched and set actions can be applied on the packet. LPTS uses the PMF architecture to classify the packet and other pre-processing attributes to apply action (punt CPU, stats and police).

### NPU and Line Card CPU in action

- In continuation of the above section, let us see how the lookup process happens in the NPU and LC CPU.
- In NPU, the ‘For Us’  control packet undergo hardware TCAM lookup which will hit one of the protocol default TCAM entries. 
- Hardware lookup result contains: 
	- listener_tag (application of interest)
    - flow type, packet destination
    - policer of the flow
    - stats pointer
- Based on lookup result, the control packets will get policed and punted to LC CPU. 
- When the ‘For Us’ packet is received by the LPTS decap node in NetIO, LPTS  does ifib lookup and find the packet associated  protocol client and deliver the packet to the protocol stack.
- This helps us to police all control traffic in hardware while performing full LPTS lookup in software before punting the packet to IOS-XR RP.

## Hardware Implementation

- The control packets, which are destined to the Route Processor (RP), are policed using a set of ingress policers in the incoming ports.
- LPTS entries in hardware TCAM classifies packets to select a policer to apply.
- LPTS has hardware policers on all default entries in the NPU to limit traffic sent to local CPU.
- Policing is done on the LC Hardware ASIC before packets hit RP/LC CPU.
- Each flow policer has default static policer rate value. 
- Each flow policer value can be conifgured to 0 (to drop all packet matching classification criteria) to max of 50K PPS.
- Similar to ASR9k platform, the LPTS policers work on a per NPU basis. For example, if the LPTS police value is set to 1000pps for a flow, it means that every NPU on the LC can punt with 1000pps to the CPU for that flow.
- L2 Protocol and Exception packets are punted to the LC via CPU traps. (need to confirm)


LPTS uses the following tuples to identify a packet:
    
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

Note: This output shows the policer values per NPU
{: .notice--info}

LPTS takes effect on all applications that receive packets from outside the router. LPTS functions without any need for customer configuration. However, the policer values can be customized if required. You should be very careful while changing this default values.

```
RP/0/RP0/CPU0:N55-24(config)#lpts pifib hardware police flow fragment rate 2000

```
<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:N55-24#show lpts pifib hardware police location all 
-------------------------------------------------------------
                Node 0/0/CPU0:
-------------------------------------------------------------
FlowType               Policer Type    Cur. Rate Burst     npu       Domain
---------------------- ------- ------- --------- --------- --------- ---------
<mark>Fragment               32102   Global  2000      197       0         0-default</mark>
OSPF-mc-known          32103   Static  2000      2000      0         0-default
OSPF-mc-default        32104   Static  100       8         0         0-default
</code>
</pre>
</div>

The below command can be used to check the locally processed packets that are accepted or dropped by the LC CPU

```
RP/0/RP0/CPU0:N55-24#show controllers npu stats traps-all instance all location all

Trap stats for lc 0


Trap Type                                     NPU  Trap TrapStats   Policer Packet               Packet

                                              ID    ID      ID              Accepted             Dropped
============================================================================================================================
RxTrapMimDiscardMacsaDrop (IRB)               0    1    0x1         32045   0                    0                    
RxTrapMimDiscardMacsaTrap (ERP_BDL)           0    2    0x2         32041   0                    0                    
RxTrapMimDiscardMacsaSnoop(dot1x)             0    3    0x3         32020   0                    0 
```


## Per Port Rate Limiting of Multicast and Broadcast Punt Packets

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
