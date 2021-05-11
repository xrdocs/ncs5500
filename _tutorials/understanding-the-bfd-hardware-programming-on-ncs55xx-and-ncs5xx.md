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

### Packet Format

Let us look at the BFD control packet format briefly. This will be useful in understanding the CLI outputs and the messages being exchanged between 2 nodes. 

![Screenshot 2021-05-11 at 12.56.15 PM.png]({{site.baseurl}}/images/Screenshot 2021-05-11 at 12.56.15 PM.png)

Apart from the above fields, we also have the optional Authentication section. For details please [refer](https://datatracker.ietf.org/doc/html/rfc5880).





