---
published: true
date: '2021-05-23 00:25 +0200'
title: NCS5500 Chassis New Generation Commons
author: Nicolas Fevrier
excerpt: >-
  New route processor, new power supply and new fan trays / fabric cards for the
  NCS 5500 chassis
position: hidden
---
{% include toc icon="table" title="New Commons for NCS5500 Chassis" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Among the innovations brought with IOS XR 7.3.1, we introduced a lot of new software features but also now hardware.  
We now support new fixed-form systems (1xRU pizza box) like: [NCS57B1-6D24 / NCS57B1-5DSE](https://www.youtube.com/watch?v=MyqmIlozL8M). And also new chassis line cards and new commons. But "commons", we mean elements like power supply, fabric cards, fan trays, route processors or system controllers (we don't actually have a new SC, but we have new generations for all others).

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/D57C38pcfyo" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}
.  

## New parts for NCS5500 Chassis

### RP2-E

This is the third generation of Route Processor, after the RP and RP-E.  
The second generation supported Class-B timing quality. And the only noticeable difference with the new RP2-E will be the supported of Class-C. To enable this feature, we will need the new RP but also line cards capable of this timing quality, for example the NC55-32T16Q4H-A (available in IOS XR 7.2.2 and 7.3.1) and the NC57-36H6D-S (coming in IOS XR 7.3.2). 

![RP2-E-01.png]({{site.baseurl}}/images/RP2-E-01.png){: .align-center}



