---
published: true
date: '2021-05-10 16:21 +0530'
title: Understanding the BFD Hardware Programming on NCS55xx and NCS5xx
author: Tejas Lad
excerpt: >-
  This document aims to cover the hardware programming of the BFD in the NCS55xx
  pipeline architecture. It also will cover the resource utilisation and  how to
  read and interpret the CLI outputs
tags:
  - NCS 5500
  - NCS 500
  - NCS5500
  - BFD
  - convergence
position: hidden
---
{% include toc icon="table" title="Understanding the BFD Hardware Programming on NCS55xx and NCS5xx" %} 

## Introduction

In our previous [artcile](https://xrdocs.io/ncs5500/tutorials/bfd-architecture-on-ncs5500-and-ncs500/), we introduced the BFD feature implementation in the pipeline architecture of NCS55xx and NCS5xx. We discussed how the packet flow and the hardware resources being utilised. We also how the scale is considered for the BFD feature and how well we have carved the resources from the ASIC with IOS-XR. In this article, we will go a bit deeper in the BFD configurations, reading the BFD outputs, hardware programming and resource utilization.

## Quick Refresh ([RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880))

As discussed in the previous article, the goal of Bidirectional Forwarding Detection (BFD) is to provide low-overhead, short-duration detection of failures in the path between adjacent forwarding engines, including the interfaces, data link(s), and, to the extent possible, the forwarding engines themselves. An additional goal is to provide a single mechanism that can be used for liveness detection over any media, at any protocol layer, with a wide range of Detection Times and overhead, to avoid a proliferation of different methods.

### BFD Control Packet Format

Let us look at the BFD control packet format briefly. This will be useful in understanding the CLI outputs and the messages being exchanged between 2 nodes. 

![Screenshot 2021-05-11 at 12.56.15 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-11 at 12.56.15 PM.png)

Apart from the above fields, we also have the optional Authentication section. For details please [refer](https://datatracker.ietf.org/doc/html/rfc5880).

| Field                         | Description                                                                                                                                                                                                                             |
|-------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Version (Vers)                | The version number of the protocol.                                                                                                                                                                                                     |
| Diagnostic (Diag)             | A diagnostic code specifying the local system's reason for the last change in session state. There are various values for the diagnostics which are defined in the RFC                                                                  |
| State (Sta)                   | This indicates the current BFD session state as seen by the transmitting system. The values are:<br>0 -- AdminDown<br>1 -- Down<br>2 -- Init<br>3 -- Up                                                                                 |
| Poll (P)                      | If set, the transmitting system is requesting verification of connectivity, or of a parameter change, and is expecting a packet with the Final (F) bit in reply.  <br>If clear, the transmitting system is not requesting verification. |
| Final (F)                     | If set, the transmitting system is responding to a received BFD Control packet that had the Poll (P) bit set.  <br>If clear, the transmitting system is not responding to a Poll.                                                       |
| Control Plane Independent (C) | If set, BFD is implemented in the forwarding plane and can continue to function through disruptions in the control plane).  <br>If clear, BFD implementation shares fate with its control plane.                                        |
| Authentication Present (A)    | If set, the Authentication Section is present and the session is to be authenticated.                                                                                                                                                   |
| Demand (D)                    | If set, Demand mode is active.<br>If clear, Demand mode is not active in the transmitting system.                                                                                                                                       |
| Multipoint (M)                | This bit is reserved for future point-to-multipoint extensions to BFD.                                                                                                                                                                  |
| Detect Mult                   | Detection time multiplier. The negotiated transmit interval, multiplied by this value, provides the Detection Time for the receiving system in Asynchronous mode.                                                                       |
| Length                        | Length of the BFD Control packet, in bytes.                                                                                                                                                                                             |
| My Discriminator              | A unique, nonzero discriminator value generated by the transmitting system, used to demultiplex multiple BFD sessions between the same pair of systems.                                                                                 |
| Your Discriminator            | The discriminator received from the corresponding remote system. This field reflects back the received value of My Discriminator, or is zero if that value is unknown.                                                                  |
| Desired Min TX Interval       | This is the minimum interval, in microseconds, that the local system would like to use when transmitting BFD Control packets.                                                                                                           |
| Required Min RX Interval      | This is the minimum interval, in microseconds, between received BFD Control packets that this system is capable of supporting.                                                                                                          |
| Required Min Echo RX Interval | This is the minimum interval, in microseconds, between received BFD Echo packets that this system is capable of supporting.                                                                                                              |

For detailed explaination of each field please [refer](https://datatracker.ietf.org/doc/html/rfc5880)


### BFD State Machine

![Screenshot 2021-05-11 at 2.24.16 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-11 at 2.24.16 PM.png)

As per the [RFC 5880](https://datatracker.ietf.org/doc/html/rfc5880), the BFD state machine is quite straightforward.  There are three states through which a session normally proceeds: two for establishing a session. They are Init and Up and one for tearing down a session. That is Down. This allows a three-way handshake for both session establishment and session teardown, assuring that both systems are aware of all session state changes.  A fourth state AdminDown exists so that a session can be administratively put down indefinitely. Each system communicates its session state in the State (Sta) field in the BFD Control packet.

| State           | Description                                                                                                                                                                                                                                                                                                                              |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Down state      | This state means that the session is down or has just been created. <br>A session remains in Down state until the remote system indicates that it agrees that the session is down by sending a BFD Control packet with the State field set to anything other than Up.                                                                    |
| Init state      | This state means that the remote system is communicating, and the local system desires to bring the session up, but the remote system does not yet realize it.  <br>A session will remain in Init state until either a BFD Control Packet is received that is signalling Init or Up state in which case the session advances to Up state |
| Up state        | This state means that the BFD session has successfully been established, and implies that connectivity between the systems is working. <br>The session will remain in the Up state until either connectivity fails or the session is taken down administratively.                                                                        |
| AdminDown state | This state means that the session is being held administratively down.  <br>This causes the remote system to enter Down state, and remain there until the local system exits AdminDown state                                                                                                                                             |

Note: For details on state machine please refer the RFC 
{: .notice--info}

