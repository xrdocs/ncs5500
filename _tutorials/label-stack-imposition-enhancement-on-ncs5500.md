---
published: true
date: '2019-12-12 16:37 +0100'
title: Label Stack Imposition Enhancement on NCS5500
author: Jose Liste
excerpt: >-
  Presentation and demonstration of the label stack imposition enhancements
  brought with IOS XR 7.0.1 on NCS5500
position: hidden
---
{% include toc icon="table" title="Label Stack Imposition Enhancement on NCS5500" %}

You can find more content related to NCS5500 including routing memory management, URPF, ACLs, Netflow following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

INTRO

## Demo/Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/watch?v=nkL301rckL4" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## Default ASIC capabilities

Shortcut in the video123: [https://youtu.be/nkL301rckL4?t=75](https://youtu.be/nkL301rckL4?t=75)

## Details on the enhancement

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=229](https://youtu.be/nkL301rckL4?t=229)

## Demo

### Test topology

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=428](https://youtu.be/nkL301rckL4?t=428)

### Service deployment with NSO

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=707](https://youtu.be/nkL301rckL4?t=707)

### SR ODN

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1030](https://youtu.be/nkL301rckL4?t=1030)

### Monitoring with Cisco Crossworks Optimization Engine (Optima)

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1372](https://youtu.be/nkL301rckL4?t=1372)

### Link failure before enhancement

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1551](https://youtu.be/nkL301rckL4?t=1551)


### Test with the improvement

```
RP/0/RP0/CPU0:Node-10#show segment-routing traffic-eng forwarding policy color 10000

SR-TE Policy Forwarding database
--------------------------------

Color: 10000, End-point: 1.1.1.2
  Name: srte_c_10000_ep_1.1.1.2
  Binding SID: 80010
  Active LSP:
    Candidate path:
      Preference: 100 (BGP)
    Local label: 80013
    Segment lists:
      SL[0]:
        Name: dynamic
        Packets/Bytes Switched: 298994357/302582289284
        Paths:
          Path[0]:
            Outgoing Label: 16007
            Outgoing Interface: GigabitEthernet0/0/0/2
            Next Hop: 11.8.10.8
            Switched Packets/Bytes: 298994357/302582289284
            FRR Pure Backup: No
            ECMP/LFA Backup: No
            Label Stack (Top -> Bottom): { 16007, 16001, 80005 }
          Path[1]:
            Outgoing Label: 16008
            Outgoing Interface: TenGigE0/0/0/3
            Next Hop: 10.6.10.6
            Switched Packets/Bytes: 0/0
            FRR Pure Backup: Yes
            ECMP/LFA Backup: No
            Label Stack (Top -> Bottom): { 16008, 16007, 16001, 80005 }

  Policy Packets/Bytes Switched: 299131510/302720539484
```

<iframe src="https://cisco.app.box.com/embed/s/ir7iwqdzu07uqh4quh57y63r8imk13vd?sortColumn=date&view=list" width="800" height="550" frameborder="0" allowfullscreen webkitallowfullscreen msallowfullscreen></iframe>

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=1936](https://youtu.be/nkL301rckL4?t=1936)

### Service removal with NSO

Shortcut in the video: [https://youtu.be/nkL301rckL4?t=2178](https://youtu.be/nkL301rckL4?t=2178)

## Recirculation impact


## Conclusion






## to delete / templates / examples

<div class="highlighter-rouge">
<pre class="highlight">
<code>CONFIG / SHOW COMMANDS / <mark>HIGHLIGHT</mark> / ETC EXAMPLE</code>
</pre>
</div>

Note to highlight: 

**Update**: blablabla
{: .notice--info}

Aligned image:  
![core-0-1-.png]({{site.baseurl}}/images/core-0-1-.png){: .align-center}
