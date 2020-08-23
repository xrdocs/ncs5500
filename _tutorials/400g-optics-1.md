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

The QSFP-DD MSA published its Revision 1.0 specifications in Sep 2016, and it is at its Revision 5.0 as of Jul 2019.

<img src="{{site.baseurl}}/images/400G/qdd-q28.png" width="450" />

### Number of Serdes

![]({{site.baseurl}}/images/400G/qdd-2-row.png){: .align-right}
QSFP-DD adds a second row of pins and increases the number of serdes from 4 to 8, hence the name double-density.

Therefore it supports optics up to 8 electrical lanes.

### 50G Serdes Support

QSFP-DD increases the maximum serdes speed from 25G to 50G, therefore supporting a maximum speed of 8x50G = 400G aggregate.

For backward compatibility, it also supports 10G and 25G serdes speed, thus allowsing all possible aggregate speeds of 40G, 100G, 200G and 400G.

Optics of all form factors, such as QSFP+, QSFP28, QSFP56, QSFP28-DD and QSFP56-DD are supported by QSFP-DD.

### PAM4 Encoding Support

QSFP-DD supports PAM4 encoding which is used by 50G serdes, while maintaining the use of NRZ encoding for 10G/25G serdes.

### RS-FEC RS(544,514) Support

QSFP-DD supports RS-FEC (Clause 91) RS(544,514) for 50G serdes, while maintaining the use of RS(528,514) for 25G serdes.

### High Power Support

QSFP-DD supports high power optics, therefore ready for coherent optics such as 400G-ZR and 400G-ZR+.

At OFC 2019, Cisco Demonstrate 20W+ power dissipation of QSFP-DD.

[https://blogs.cisco.com/sp/cisco-demonstrates-20w-power-dissipation-of-qsfp-dd-at-ofc-2019](https://blogs.cisco.com/sp/cisco-demonstrates-20w-power-dissipation-of-qsfp-dd-at-ofc-2019)

[https://www.lightwaveonline.com/optical-tech/transmission/article/14036073/qsfpdd-msa-releases-common-management-interface-40-and-hardware-specification-50](https://www.lightwaveonline.com/optical-tech/transmission/article/14036073/qsfpdd-msa-releases-common-management-interface-40-and-hardware-specification-50)


### Flexible Breakout Support

The wide range of combination of serdes number and speeds provides very flexible breakout options:

<style>
table foo {
  width: auto;
}

<div class="foo">

 QSFP56-DD | QSFP28-DD | QSFP56 | QSFP28 | QSFP+ 
:---------:|:---------:|:------:|:------:|:-----:
 400G      | 200G      | 200G   | 100G   | 40G   
 2x200G    | 2x100G    | 2x100G | 4x25G  | 4x10G 
 4x100G    | 8x25G     | 4x50G  |        |       
 8x50G     |           |        |        |       

</div>

