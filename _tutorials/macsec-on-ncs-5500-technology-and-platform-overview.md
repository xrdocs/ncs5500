---
published: true
date: '2020-04-30 13:52 -0700'
title: MACsec on NCS-5500 - Technology and Platform Overview
author: Vincent Ng
excerpt: MACsec on NCS-5500 - Technology and Platform Overview
position: top
tags:
  - iosxr
  - MACsec
  - NCS-5500
---
{% include toc %}

## MACsec on NCS 5500 - Technology and Platform Overview

### Introduction
This document is the first part of a series, and provides an overview of MACsec technology, data plane overhead, basic configuration and platform support. MACsec is a line-rate Ethernet encryption and works at Layer 2, with hop-by-hop links.  MACsec is based on IEEE standards, and is supported in Cisco’s NCS-5500 and many other Cisco Platforms. MACsec products based on IEEE MKA standards will interop with each other seamlessly.


### MACsec Technology Overview and Benefits

MACsec is a Layer 2 IEEE 802.1AE standard for encrypting packets between two MACsec-capable routers. It supports IEEE 802.1AEbn 256bit encryption and uses Advanced Encryption Standard (AES) algorithm. MACsec secures data on physical media, making it impossible for data to be compromised at higher layers. Security breaches can occur in any layer and MACsec prevents from layer 2 security breaches, including Packet sniffing, Packet Eavesdropping, DOS attack, tampering, MAC address spoofing and ARP spoofing etc. 

Some of the major MACsec benefits are,
- **Confidentiality:** MACsec helps ensure confidentiality by providing strong encryption at layer 2
- **Integrity:** MACsec provides integrity checking to help ensure that data cannot be modified in transit
- **Flexibility:** You can selectively enable MACsec on per interface basis by attaching MACsec policy which gives flexibility for having both Secured (MACsec enabled) and Non secured ports to operate on same Router 
- **Network Intelligence:** Unlike end-to-end, Layer 3 encryption techniques that hide the contents of packets from the network devices they cross, MACsec encrypts packets on a hop-by-hop basis at Layer 2, allowing the network to inspect, monitor, mark, and forward traffic according to your existing policies

As MACsec is the hop-by-hop encryption technology, Frames gets encrypted as it leaves the wire (PHY or FPGA post NPU operation) and gets decrypted before it ingress to NPU. So NPU will have complete view of data and will be able to provide any services required to these packets. 

MACsec allows you to secure an Ethernet link including all control plane protocol packets except EAPoL. It uses the IEEE 802.1X MACsec Key Agreement protocol (MKA) to exchange session keys, and manage encryption keys.


### Commonly used MACsec Terminologies

**MACsec Key Agreement (MKA)** is defined in IEEE 802.1X is a key agreement protocol for discovering MACsec peers and negotiating keys  .

**Secure Channel (SC)** is a security relationship used to provide security guarantees for frames transmitted from one member of a CA to the others. An SC is supported by a sequence of SAs thus allowing the periodic use of fresh keys without terminating the relationship.  

**Secure Channel Identifier (SCI)** is a globally unique identifier for a secure channel, comprising a globally unique MAC Address and a Port Identifier, unique within the system allocated that address.  

**Connectivity Association Key (CAK)** is a long-lived master key used to generate all other keys used for MACsec. In our implementation, it is the Pre-Shared Key (PSK) configured through a key chain. CAK is a hex string of 16 bytes for AES 128bit cipher and 32 bytes for 256bit cipher.  

**CAK Key Name (CKN)** It is used to identify the CAK. It is a hex string of 1 to 32 bytes. CKN has to be same on both side to form session successfully.  

**Secure Association Key (SAK)** is derived by the elected Key Server from the CAK and SAK is used by the router/end devices to encrypt traffic and decrypt traffic for a given session.  

**Key Sever Priority** is the optional value, which can be configured in MACsec policy.  

**Key server (KS)** is the one, which controls key generation and distribution of SAK to clients (Non-KS). Device with lowest key server priority value preferred to win key server election. In case of tie, lowest value of SCI wins.


### MACsec Data Plane
Once we enable MACsec on a link, Both Tx and Rx SCI (Secure Channel Identifier) and associated Tx & Rx SAs which is distributed by Key Server gets programmed in hardware. Any traffic leaving the interface will get encrypted using Tx SA policy programmed in hardware only with exception of EAPoL packets as this takes the different path inside MACsec core (Clear Path). And traffic getting ingress to the interface will gets decrypted using the programmed Rx SA policy. Once SCI gets programmed for that interface, MACsec policy gets pushed to the hardware which enables the interface to apply access-control policy (should / must secure) for all traffics leaving interface till SA gets programmed.  

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-tag-format.png?raw=true)

MACsec inserts two tags for all data frames, which egress the interface. Which are SecTag and ICV. The value of these additional overheads can be from 16 to 32Byte maximum.  Both SecTag and ICV can vary from 8B to 16B depending upon the information it carries and cipher it uses. SecTag carry an 8 byte SCI that is optional. The authentication is provided to the complete frame except CRC and ICV part, which resides at end of the frame. And Encryption is provided starting from VLAN header (if used) till Payload.

Cisco’s implementation always uses 16B SecTag and 16B ICV, so the data plane overhead is 32B.

### Use Cases
One common use case of NCS-5500 can be link MACsec on all regular IP core / MPLS core devices which are generally part of service provider network. MACsec can simply be enabled on all back to back links over IP/MPLS core devices as an underlay protocols. This will still get you the high-speed lean core network with complete security provided by MACsec encryption as MACsec works on wire speed.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-reference-topology.png?raw=true)

As you can see in above figure, MACsec is enabled between each links connected between each core devices on your WAN from PE to PE. Take a look at below figure, which gives the comparison of different possible frames over IP/MPLS network in both clear and encrypted format. 
As you can see, Encryption starts right after Source MAC address and ends just before FCS in all kinds frames gets into IP/MPLS core.  
MACsec over bundle is supported on NCS55xx family of products. MACsec is enabled on all bundle member interfaces individually and we will have separate sessions for each member as MACsec works on MAC layer.  Since MACsec is enabled on per member interface basis, we can have bundle, which contains mixing of MACsec and Non MACsec enabled links as member of same bundle interface.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-frame-format.png?raw=true)

### MACsec Basic Configuration
MACsec can be configured in 3 simple steps:
1.         Create Key Chain (to configure the PSK - CKN & CAK)
2.         Create MACsec policy (optional, to configure encryption cipher & other policies etc.)
3.         Attach created key chain and policy to an interface.

**key chain** *psk_name*  
&nbsp;**macsec**  
&nbsp;&nbsp;**key** *ckn-2-to-64-hex-char*  
&nbsp;&nbsp;&nbsp;**key-string** *cak-32-or-64-hex-char* **cryptographic-algorithm** {**aes-128-cmac|aes-256-cmac**}  
&nbsp;&nbsp;&nbsp;&nbsp;**lifetime** *start-time start-date* {*end-time end-date*|**duration** *seconds*|**infinite**}  

**macsec-policy** *policy_name*  
&nbsp;\[optional-policies\]

**interface** *Interface_name*  
&nbsp;**macsec psk-keychain** *psk_name* \[**policy** *policy_name*\]

  
A basic MACsec configuration, with default policy GCM-AES-XPN-256:
<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
key chain psk1
 macsec
  key 01
   key-string 0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef cryptographic-algorithm aes-256-cmac
   lifetime 00:00:00 january 01 2020 infinite
  !
 !
!
interface HundredGigE0/0/2/0
 macsec psk-keychain psk1
!
      </code>
      </pre>
      </div>

<div class="highlighter-rouge">
      <pre class="highlight">
      <code>
RP/0/RP0/CPU0:55A2-MOD-SE-6625#show macsec mka summary

NODE: node0_0_CPU0
========================================================================================
   Interface-Name     Status     Cipher-Suite       KeyChain       PSK/EAP      CKN
========================================================================================
     Hu0/0/2/0        Secured  GCM-AES-XPN-256        psk1         PRIMARY      01

Total MACSec Sessions : 1
     Secured Sessions : 1
     Pending Sessions : 0
      </code>
      </pre>
      </div>
  
### MACsec Platform Support
MACsec is supported on both modular and fixed platforms. However, on some platforms, not all ports will support MACsec. Below is a summary of the platforms with support ports highlighted.
  
**NC55-36x100G-S MACsec Modular Line Card**
All 36x100G ports support MACsec
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-1.png?raw=true)
  
  
**NC55-6x200-DWDM-S IPoDWDM Modular Line Card**
All 6x100G/200G ports support MACsec
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-2.png?raw=true)
  
  
**NC55-MOD-A(-SE)-S MOD Line Card with MPA**
All 12x10G, 2x40G ports and both MPA support MACsec
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-3.png?raw=true)
  
  
**NCS-55A1-36H(-SE)-S Fixed Chassis**
All 36x100G ports support MACsec
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-4.png?raw=true)
  
  
**NCS-55A2-MOD(-SE)-S MOD Fixed Chassis with MPA**
The 16x25G ports and both MPA support MACsec, while the 24x10G ports do not.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-5.png?raw=true)
  
  
**NCS-55A1-24Q6H-S Fixed Chassis**
The 6x100G and 16 out of 24x25G ports support MACsec, while the 24x10G and 8 out of 24x25G ports do not.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-6.png?raw=true)
  
  
**NCS-55A1-48Q6H Fixed Chassis**
The 6x100G ports support MACsec, while the 48x25G ports do not.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-7.png?raw=true)
  
  
**NCS-55A1-24Q6H-SS Fixed Chassis**
All 6x100G ports, 24x25G ports and 24x10G ports support MACsec.
![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/MACsec-platform-9.png?raw=true)
  
  
**NC55-MPA-2TH-S NC55-MPA-1TH2H-S NC55-MPA-4H-S NC55-MPA-12T-S**
All MPA ports, 10G, 100G and CFP2 support MACsec.  
<img src="{{site.baseurl}}/images/evpn-config/MACsec-platform-8.png" width="400" />
  
  
**Platform Matrix for MACsec Support**  

|          Platform        |          SFP+   |          SFP28 |          QSFP+            |          QSFP28          |            CFP2   |
|          :-----    |          :-----    |          :-----    |          :-----    |          :-----    |          :-----    |
|          NC55-36x100G-S    |          -           |          -           |          -           |          10G/40G/4x10G/100G            |          -           |
|          NC55-6x200-DWDM-S      |          -           |          -           |          -           |          -           |            Nx100G         |
|          NC55-MOD-A(-SE)-S          |          10G    |          -           |          4x10G/40G   |          -           |            -           |
|          NCS-55A1-36H(-SE)-S        |          -           |          -           |          -           |            10G/40G/4x10G/4x25G/100G   |          -           |
|          NCS-55A2-MOD(-SE)-S      |          -           |          10G/25G       |          -           |          -           |            -           |
|          400G MPA’s |          10G    |          -           |          -           |          40G/4x10G/4x25G/100G |            Nx100G         |
|          NCS-55A1-48Q6H   |          -           |          -           |          -           |            40G/4x10G/4x25G/100G |          -           |
|          NCS-55A1-24Q6H-S           |          -           |          10G/25G       |          -           |            40G/4x10G/4x25G/100G |          -           |
|          NCS-55A1-24Q6H-SS          |          10G         |          10G/25G       |          -           |            40G/4x10G/4x25G/100G |          -           |

Please note there is no 1G or 100M support for MACsec.
