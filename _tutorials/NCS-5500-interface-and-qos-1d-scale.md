---
published: true
date: '2021-03-18 21:23 -0700'
title: NCS-5500 Interface and QoS 1D Scale
author: Vincent Ng
excerpt: NCS-5500 Interface and QoS 1D Scale
position: hidden
---
{% include toc %}

## Introduction

This tutorial will discuss single dimensional interface and QoS scale of the NCS-5500 series.

The first section will introduce the basic L2 and L3 interface scale, without enabling QoS.

The next section will discuss the impact of HQOS mode on bundle interface scale. HQOS mode is required if you need to enable Egress QoS on subinterfaces.

Then we will discuss the impact of enabling QoS on interface scale, with respect to both ingress and egress.

## Non-QoS L2 and L3 Interface Scale 

Below interface scales are effective from IOS XR 6.6.3 and 7.0.1. No QoS and no HQOS mode for these scales.

### L3 Interface and Subinterface Scale

L3 subinterfaces scale is both per interface and per system, relevant limits are:

L3 subinterfaces <= 2000  
L3 bundle subinterfaces <= 1024  
L3 subinterfaces + L3 bundle subinterfaces <= 2000  
L3 bundle main interfaces + L3 bundle subinterfaces <= 1790  
L3 main interfaces + L3 subinterfaces < 2558  
L3 (main + subinterfaces) + L3 bundle (main+subinterfaces) < 2558  

### L2 Interface and Subinterface Scale

Max subinterfaces per interface is 4094 (0,4095 reserved), relevant limits are:

L2 subinterfaces <= 4094/4095 per interface/system  
L2 (main + subinterfaces) <= 4094/4095 per system  
L2 bundle subinterfaces <= 4094/4096 per interface/system  
L2 (bundle main + bundle subinterfaces) <= 4097 per system  
L2 (subinterfaces + bundle subinterfaces) <= 8191 per system  
L2 (main + subinterfaces + bundle main + bundle subinterfaces) <=8192 per system  

### Mixed L2 and L3 Interface and Subinterface Scale

For mixed L2 and L2, relevant limits are:

(L3 + L2) subinterfaces <= 6095 system  
(L3 + L2) main interfaces <= 2304 system  
(L3 + L2) main + subinterfaces <= 6652 per system  
(L3 + L2) subinterfaces + bundle subinterfaces <= 8192 system  

### Non-QoS Bundle Interface Scale

Bundle interface scale is configurable in IOS XR, with less bundle members support for higher bundle interface scale.

Let N be the maximum number of bundle interfaces supported. Default N is 256, and can be changed by:

```
RP/0/RP0/CPU0:OC-SE(config)#hw-module profile bundle-scale ?
  1024  Max 1024 trunks, Max 16 members (128 main + 896 sub)
  512   Max 512 trunks, Max 32 members  (128 main + 384 sub)
  256   Max 256 trunks, Max 64 members  (128 main + 128 sub)
```




<style>
 .table1 {
  table-layout: fixed;
  width: 50%;
  text-align: center;
  color: black;
}
 .table2 {
  table-layout: fixed;
  width: 90%;
  text-align: center;
  color: black;
}
 .head1 { background: lightskyblue;}
 .head2 { background: lightskyblue;}
 .body1 { background: green;}
</style>
<div class="table1">
<table>
  <colgroup>
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
  </colgroup>
  <tr>
    <td class="head1" colspan="5">Non-QoS Bundle Scale N (256/512/1024)</td>
  </tr>
  <tr>
    <td colspan="2">Main</td>
    <td colspan="2">Subinterface</td>
    <td rowspan="2">Total Scale</td>
  </tr>
  <tr>
    <td>L2</td>
    <td>L3</td>
    <td>L2</td>
    <td>L3</td>
  </tr>
  <tr>
    <td class="head2" colspan="5">HQOS Mode</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td>N-1</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>N-1</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>N-1</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>N</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td>N</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>N</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>N</td>
  </tr>
  <tr>
    <td class="head2" colspan="5">Non-HQOS Mode</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>4096</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>4096</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td>1790</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>4097</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td>5120</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>5120</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>5120</td>
  </tr>
</table>
</div>
<div class="table2">
<table>
  <colgroup>
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
    <col width="10%" />
  </colgroup>
  <tr>
    <td class="head1" colspan="9">Non-QoS L2 and L3 Interface Scale</td>
  </tr>
  <tr>
    <td colspan="4">Physical Interfaces</td>
    <td colspan="4">Bundle Interfaces</td>
    <td rowspan="3">Total Scale</td>
  </tr>
  <tr>
    <td colspan="2">Main</td>
    <td colspan="2">Subinterface</td>
    <td colspan="2">Main</td>
    <td colspan="2">Subinterface</td>
  </tr>
  <tr>
    <td>L2</td>
    <td>L3</td>
    <td>L2</td>
    <td>L3</td>
    <td>L2</td>
    <td>L3</td>
    <td>L2</td>
    <td>L3</td>
  </tr>
  <tr>
    <td class="head2" colspan="9">Non-HQOS Bundle interfacecs only</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>4096</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>4096</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td>1024</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td>1790</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>4097</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td>5120</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>5120</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>5120</td>
  </tr>
  <tr>
    <td class="head2" colspan="9">Physical Interfaces only</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>2000</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>4095</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>6095</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>2304</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>2304</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>2304</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>2557</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>4095</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>6399</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>6652</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td>6652</td>
  </tr>
  <tr>
    <td class="head2" colspan="9">Non-HQOS Mixed Physical and Bundle Interfaces</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td>2000</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>8191</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td>5119</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>6096</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td></td>
    <td></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>8192</td>
  </tr>
  <tr>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td>2557</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td class="body1"></td>
    <td></td>
    <td>8192</td>
  </tr>
  <tr>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td class="body1"></td>
    <td>8192</td>
  </tr>
</table>
</div>
