---
published: true
date: '2020-06-16 19:18 -0700'
title: NCS-5700 400G Optics Technology and Roadmap
author: Vincent Ng
excerpt: NCS-5700 400G Optics Support
tags:
  - iosxr
  - NCS-5700
  - 400G
  - Optics
position: hidden
---
## NCS-5700 400G Optics Technology and Roadmap

NCS-5700 is the first platform in the NCS-5500 Series that supports 400G optics, which is introduced in a previous tutorial:

[Introducing 400GE on NCS5500 Series](https://xrdocs.io/ncs5500/tutorials/introducing-400ge-on-ncs5500-series/)

This tutorial will discuss in more details the 400G optics technology and roadmap for the NCS-5700 series.

A follow on tutorial will discuss the 400G optics and breakout options supported for each port type on the NCS-5700 series linecards, NC57-24DD and NC57-18DD-SE.


## Introducing QSFP-DD MSA

QSFP-DD is the leading form factor for 400G optics, promoted by QSFP-DD [MSA](http://www.qsfp-dd.com/). Members include Cisco and other major suppliers.

QSFP+ and QSFP28 are the de-facto standard for high density 40G and 100G optics.

QSFP-DD is fully compatible with QSFP+ and QSFP28, and so maintains the same port density.

![]({{site.baseurl}}/images/400G/qdd-q28.png)

### Number of Serdes

![]({{site.baseurl}}/images/400G/qdd-2-row.png){: .align-right}
QSFP-DD adds a second row of pins and increases the number of serdes from 4 to 8, hence the name double-density.

Therefore it supports optics up to 8 electrical lanes.

### 50G Serdes Support

QSFP-DD increases the maximum serdes speed from 25G to 50G, therefore supporting a maximum aggregate speed of 8x50G = 400G aggregate.

For backward compatibility, it also supports 10G and 25G serdes speed, thus allowsing all possible aggregate speeds of 40G, 100G, 200G and 400G.

Optics of all form factors, such as QSFP+, QSFP28, QSFP56, QSFP28-DD and QSFP56-DD are supported by QSFP-DD.

##


