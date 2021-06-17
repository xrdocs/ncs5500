---
published: true
date: '2021-03-18 21:23 -0700'
title: NCS-5500 Interface and QoS 1D Scale
author: Vincent Ng
excerpt: NCS-5500 Interface and QoS 1D Scale
position: top
---
{% include toc %}

## Introduction

This tutorial will discuss single dimensional interface and QoS scale of the NCS-5500 series.

The first section will introduce the basic L2 and L3 interface scale, without enabling QoS.

The next section will discuss the impact of HQOS mode on bundle interface scale.

Then we will discuss the impact of enabling QoS on interface scale, with respect to both ingress and egress.

## Non-QoS L2 and L3 Interface Scale 

Below interface scales are effective from IOS XR 6.6.3 and 7.0.1. No QoS and no HQOS mode for these scales. Maximum bundle interface scale is assumed to be the maximum 1024, and more details will be discussed on bundle scale in the next section.

### L3 Interface and Subinterface Scale

For L3 main interfaces, it is the number of physical interfaces present on a particular fixed or modular chassis, whatever the speed. Please note for breakout, it is counted as the number of interfaces after the breakout.

L3 subinterfaces scale is both per interface and per system, relevant limits are:

L3 subinterfaces <= 2000  
L3 bundle subinterfaces <= 1024  
L3 subinterfaces + L3 bundle subinterfaces <= 2000  
L3 bundle main interfaces + L3 bundle subinterfaces <= 1790  
L3 main interfaces + L3 subinterfaces < 2558  
L3 (main + subinterfaces) + L3 bundle (main + subinterfaces) < 2558  

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
(L3 + L2) subinterfaces + bundle subinterfaces <= 8192 per system  
(L3 + L2) maian and subinterfaces + bundle main and subinterfaces <= 10496 per system

Below table will present the above scale limits in graphical format for ease of visualization:

<style>
  .table1 tr {line-height: 100%}
  .table1 tr:first-of-type {font-weight: bold;}
  .table1 {
   table-layout: fixed;
   width: 90%;
   text-align: center;
   color: black;
}
 .table2 tr {line-height: 100%}
  .table2 tr:first-of-type {font-weight: bold;}
 .table2 {
  table-layout: fixed;
  width: 50%;
  text-align: center;
  color: black;
}
 .head1 { background: lightgrey;}
 .head2 { background: whitesmoke;}
 .body1 { background: lightgreen;}
</style>
<div class="table1">
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
    <td class="head2" colspan="9">Non-HQOS Bundle interfaces only</td>
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
    <td>10496</td>
  </tr>
</table>
</div>

## Non-QoS Bundle Interface Scale

Bundle interface scale is configurable in IOS XR, with less bundle members support for higher bundle interface scale.

Let N be the maximum number of bundle interfaces supported. Default N is 256, and can be changed by:
```
RP/0/RP0/CPU0:NCS5500(config)#hw-module profile bundle-scale ?
  1024  Max 1024 trunks, Max 16 members (128 main + 896 sub)
  512   Max 512 trunks, Max 32 members  (128 main + 384 sub)
  256   Max 256 trunks, Max 64 members  (128 main + 128 sub)
```
### Non-HQOS Mode Bundle Interface Scale

The bundle interface scale is independent of N for non-HQOS mode, and relevant limits are:

L3 Bundle subinterfaces <= 1024 per system  
L2 Bundle subinterfaces <= 4094/4096 per interface/system  
L2 Bundle subinterfaces + L3 Bundle subinterfaces <= 4096 per system  
L2 Bundle main interfaces + L3 Bundle main interfaces <=1024  
L2 Bundle main interfaces + L2 Bundle subinterfaces <=4097  
L2 Bundle main interfaces + L3 Bundle subinterfaces <=2047  
(L2 + L3) Bundle main interfaces + (L2 + L3) Bundle subinterfaces <= 5120  

### HQOS mode Bundle Interface Scale

HQOS mode is needed to enable egress hierarchical QoS policies on main interfaces, or to enable egress flat or hierarchical QoS policies on subinterfaces.

HQOS mode is disabled by default, and can be enabled by below command:
```
hw-module profile qos hqos-enable
```
HQOS mode will impose additional limits on bundle interface scale, depending on N:

L3 Bundle subinterfaces + L2 Bundle subinterfaces < N  
L3 Bundle main interfaces + L2 Bundle main interfaces <= N  
(L2 + L3) Bundle main interfaces + (L2 + L3) Bundle subinterfaces <= N  

Below table will present the above scale limits in graphical format for ease of visualization:

<div class="table2">
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
</table>
</div>

## Ingress and Egress QoS Interface Scale

Enabling QoS will have impact on interface scale, and the effect will be different for ingress and egress, due to the difference in hardware resources required.

### Ingress QoS Interface Scale

Ingress QoS scale is impacted by the available counters for statistics.

In Normal QoS mode, 2 counters are used per ingress policy-map, and support the best interface scale.

If Enhanced QoS mode is enabled, 4 counters are used per ingress policy-map, providing better statistics, but also used more hardware resources, resulting in lower interface scale:
```
hw-module profile stats qos-enhanced
```
The default maximum number of class-maps per ingress policy-map is 32. If you configure a smaller max-classmap-size, it will result in higher interface scale:
```
hw-module profile qos max-classmap-size [4|8|16|32]
```
The maximum number of unique ingress policy-maps per NPU is increased from 30 to 250 from IOS XR 6.6.3/7.0.1. However, this does not impact on the ingress QoS interface scale.

HQOS mode is not required for ingress qos, even with hierarchical policy-maps, and HQOS mode does not impact ingress QoS interface scale.

A main interface consume the same resources as a subinterface with ingress QoS, so we would just refer to number of interfaces for scale purpose.

For each bundle member with ingress QoS on a core of a NPU,  QoS resources are consumed on both of the 2 cores of that NPU, so per core and per NPU interface scale are the same.

Therefore a bundle main/subinterface with M bundle members will consume the equivalent resources of 2xM interfaces with ingress QoS.

Below table is the number of interfaces with ingress policy-map attached:

<style>
.ingressqos table, th {text-align: center;}
.ingressqos table th:first-of-type {width: 15%;}
.ingressqos table th:nth-of-type(2) {width: 20%;}
.ingressqos table th:nth-of-type(3) {width: 20%;}
.ingressqos table th:nth-of-type(4) {width: 22%;}
.ingressqos table th:nth-of-type(5) {width: 23%;}
</style>
<div class="ingressqos" markdown="1">

| QoS Mode | Class-Map Size | Scale per Core | Scale per NPU   Bundle | Scale per NPU   Physical |
|----------|----------------|----------------|------------------------|--------------------------|
| Normal   | 4              | 1023           | 1023                   | 2046                     |
| Normal   | 8              | 511            | 511                    | 1022                     |
| Normal   | 16             | 255            | 255                    | 510                      |
| Normal   | 32             | 127            | 127                    | 254                      |
| Enhanced | 4              | 871            | 871                    | 1742                     |
| Enhanced | 8              | 435            | 435                    | 870                      |
| Enhanced | 16             | 217            | 217                    | 434                      |
| Enhanced | 32             | 108            | 108                    | 216                      |

</div>

### Egress QoS Interface Scale

Egress QoS scale is impacted by the available queues (VOQ Virtual Output Queues).

In NCS-5500 architecture, the egress queues are actually mapped to the buffer on ingress, and organized as virtual output queues. The benefits of this approach are minimized egress buffers, and allows single lookup on ingress for forwarding. 

In this model, the ingress NPU has to store information for all egress NPU/Linecard, so scale will vary with number of egress NPU/Linecards.

Egress qos policy-map supports a maximum of 8 class-maps and each with 1 queue, by default, and no configuration is required.

Each physical interface will always be assigned 8 queues, which is fixed.

Each physical subinterface will be assigned 8 additional queues when it is created and with egress QoS configured.

Bundle main interface with M bundle members does not consume additional queues other than the queues of the M bundle members.

Each bundle member of a bundle subinterface will be assigned 8 additional qeueus when the bundle subinterface is created and with egress QoS configured.

Therefore a bundle subinterface with M bundle members will consume the equivalent resources of M interfaces with egress QoS.

HQOS mode is required if you use egress hierarchical policy-maps on a main/subinterface, and also egress flat policy-map on a subinterface. HQOS mode will not impact egress QoS interface scale, but will impact bundle interface scale as in above section.

Below table is the number of main/subinterfaces with egress policy-map attached:

<style>
.egressqos table, th {text-align: center; width: 70%}
.egressqos th:first-of-type {width: 20%;}
.egressqos th:nth-of-type(2) {width: 10%;}
.egressqos th:nth-of-type(3) {width: 10%;}
.egressqos th:nth-of-type(4) {width: 10%;}
.egressqos th:nth-of-type(5) {width: 10%;}
</style>
<div class="egressqos" markdown="1">

| Scale   per Core      | Fixed | 5504 | 5508 | 5516 |
|-----------------------|-------|------|------|------|
| Before   IOS XR 7.0.1 | 512   | 48   | 48   | 48   |
| IOS XR   7.0.1        | 1024  | 192  | 96   | 48   |

</div>
