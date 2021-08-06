---
published: true
date: '2021-08-06 14:14 +0530'
title: Coexistence Between BFD over Bundle and BFD over Logical Bundle
position: hidden
author: Tejas Lad
excerpt: >-
  This document will cover the Coexistence Between BFD over Bundle and BFD over
  Logical Bundle and why it is needed
tags:
  - iosxr
  - BFD
  - BLB
  - BOB
  - NCS 5500
  - NCS 500
  - BLB BoB Coexistence
---
{% include toc icon="table" title="Coexistence Between BFD over Bundle and BFD over Logical Bundle" %} 

## Introduction

In the previous articles, we introduced the concepts of [BFD over Bundle](https://xrdocs.io/ncs5500/tutorials/bfd-over-bundle-interfaces-on-ncs5500-and-ncs500/) and [BFD over Logical Bundle](https://xrdocs.io/ncs5500/tutorials/multipath-and-multihop-bfd-sessions-on-ncs5500-and-ncs500/). We saw the configurations and the use cases. In this article we will discuss the limitations of both and see the why we need their coexistence.

## Quick recap

![Screenshot 2021-08-06 at 2.45.40 PM.png]({{site.baseurl}}/images/Screenshot 2021-08-06 at 2.45.40 PM.png)


BFD over Bundle(BoB) implementation is a standard based fast failure detection of link aggregation (LAG) member links that is interoperable between different platforms. BFD over Bundle implements BFD per member link. Whereas the Bidirectional Forwarding Detection (BFD) over Logical Bundle feature implements and deploys BFD over bundle interfaces. This is the fundamental difference between BLB and BoB. In the former the bundle interface is a single interface, whereas in the later we implement BFD per member link. BLB is a multipath (MP) single-hop session.If BLB is running on a bundle there is only one BFD session running. This implies that only one bundle member is being monitored by BFD, at any given time. Whereas in case of BoB we will BFD sessions equal to the number of member links.

## Limitations 

  - BoB does not provide true L3 check and is not supported on subinterfaces.
  - With BLB, a failure of bundle members, which BFD is not running on is not detected.
  - And a failure of a bundle member, which BFD is running on will cause BFD to declare a session     failure on the bundle, even if there are sufficient numbers of other bundle members available     and functional.


