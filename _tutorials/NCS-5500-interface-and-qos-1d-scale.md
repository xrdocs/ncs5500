---
published: true
date: '2021-03-18 21:23 -0700'
title: NCS-5500 Interface and QoS 1D Scale
author: Vincent Ng
excerpt: NCS-5500 Interface and QoS 1D Scale
position: hidden
---
{% include toc %}

## A New Post

<style>
 .table1 {
  table-layout: fixed;
  width: 50%;
  text-align: center;
}
 .table2 {
  table-layout: fixed;
  width: 90%;
  text-align: center;
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
    <td class="head1" colspan="4">Bundle Scale N (256/512/1024)</td>
    <td rowspan="3">Total Scale</td>
  </tr>
  <tr>
    <td colspan="2">Main</td>
    <td colspan="2">Subinterface</td>
  </tr>
  <tr>
    <td>L2</td>
    <td>L3</td>
    <td>L2</td>
    <td>L3</td>
  </tr>
  <tr>
    <td class="head2" colspan="4">HQOS Mode</td>
    <td></td>
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
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>
</div>
