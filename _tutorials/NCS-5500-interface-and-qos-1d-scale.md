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

This tutorial will discuss some single dimensional scale of the NCS-5500 series.

The first section will introduce the basic L2 and L3 physical/bundle/main/subinterface scale, without enabling QoS.

The next section will discuss the impact of HQOS mode on bundle interface scale, still without enabling QoS. HQOS mode is required if you require Egress QoS on subinterfaces.

Then we will discuss


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
