---
published: true
date: '2020-04-26 22:35 -0700'
title: 'MACsec on NCS 5500 - Technology and Platform Overview '
author: Vincent Ng
position: hidden
---
{% include toc %}

## MACsec on NCS 5500 - Technology and Platform Overview

### Introduction
This document is the first part of a series, and provides an overview of MACsec technology, data plane overhead, basic configuration and platform support. MACsec is a line-rate Ethernet encryption and works at Layer 2, with hop-by-hop links.  MACsec is based on IEEE standards, and is supported in Cisco’s NCS-5500 and many other Cisco Platforms. MACsec products based on IEEE MKA standards will interop with each other seamlessly.


### MACsec Technology Overview and Benefits

MACsec is a Layer 2 IEEE 802.1AE standard for encrypting packets between two MACsec-capable routers. It supports IEEE 802.1AEbn 256bit encryption and uses Advanced Encryption Standard (AES) algorithm. MACsec secures data on physical media, making it impossible for data to be compromised at higher layers. Security breaches can occur in any layer and MACsec prevents from layer 2 security breaches, including Packet sniffing, Packet Eavesdropping, DOS attack, tampering, MAC address spoofing and ARP spoofing etc. 

Some of the major MACsec benefits are,
- Confidentiality: MACsec helps ensure confidentiality by providing strong encryption at layer 2
- Integrity: MACsec provides integrity checking to help ensure that data cannot be modified in transit
- Flexibility: You can selectively enable MACsec on per interface basis by attaching MACsec policy which gives flexibility for having both Secured (MACsec enabled) and Non secured ports to operate on same Router 
- Network Intelligence: Unlike end-to-end, Layer 3 encryption techniques that hide the contents of packets from the network devices they cross, MACsec encrypts packets on a hop-by-hop basis at Layer 2, allowing the network to inspect, monitor, mark, and forward traffic according to your existing policies

As MACsec is the hop-by-hop encryption technology, Frames gets encrypted as it leaves the wire (PHY or FPGA post NPU operation) and gets decrypted before it ingress to NPU. So NPU will have complete view of data and will be able to provide any services required to these packets. 

MACsec allows you to secure an Ethernet link including all control plane protocol packets except EAPoL. It uses the IEEE 802.1X MACsec Key Agreement protocol (MKA) to exchange session keys, and manage encryption keys.


### Commonly used MACsec Terminologies

**MACsec Key Agreement (MKA)** is defined in IEEE 802.1X is a key agreement protocol for discovering MACsec peers and negotiating keys  .

**Secure Channel (SC)** is a security relationship used to provide security guarantees for frames transmitted from one member of a CA to the others. An SC is supported by a sequence of SAs thus allowing the periodic use of fresh keys without terminating the relationship.  

**Secure Channel Identifier (SCI)** is a globally unique identifier for a secure channel, comprising a globally unique MAC Address and a Port Identifier, unique within the system allocated that address.  

**Connectivity Association Key (CAK)** is a long-lived master key used to generate all other keys used for MACsec. In our implementation, it is the Pre-Shared Key (PSK) configured through a key chain. CAK is of 16 byte or 32 byte length for AES 128bit cipher and 64 character for 256bit cipher.  

**CAK Key Name (CKN)** It is used to identify the CAK. Length is of 1 to 16 byte. CKN has to be same on both side to form session successfully.  

**Secure Association Key (SAK)** is derived by the elected Key Server from the CAK and SAK is used by the router/end devices to encrypt traffic and decrypt traffic for a given session.  

**Key Sever Priority** is the optional value, which can be configured in MACsec policy.  

**Key server (KS)** is the one, which controls key generation and distribution of SAK to clients (Non-KS). Device with lowest key server priority value preferred to win key server election. In case of tie, highest value of SCI wins.


### MACsec Data Plane
Once we enable MACsec on a link, Both Tx and Rx SCI (Secure Channel Identifier) and associated Tx & Rx SAs which is distributed by Key Server gets programmed in hardware. Any traffic leaving the interface will get encrypted using Tx SA policy programmed in hardware only with exception of EAPoL packets as this takes the different path inside MACsec core (Clear Path). And traffic getting ingress to the interface will gets decrypted using the programmed Rx SA policy. Once SCI gets programmed for that interface, MACsec policy gets pushed to the hardware which enables the interface to apply access-control policy (should / must secure) for all traffics leaving interface till SA gets programmed.  