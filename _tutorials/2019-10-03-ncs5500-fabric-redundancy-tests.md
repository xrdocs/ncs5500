---
published: false
date: '2019-10-03 08:52 +0200'
title: NCS5500 Fabric Redundancy Tests
---
{% include toc icon="table" title="NCS5500 Fabric Redundancy Tests" %} 

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Third episode of the lab series, today, we will talk about NCS5500 fabric. More specifically, we will try to qualify the impact of losing a fabric card in a chassis.

As you certainly know, the NCS5500 exists in fixed systems and modular chassis. For this second category, we have 4-slot, 8-slot and 16-slot versions.  



If they can all mix and match different type of line cards, each chassis type is using specific fabric cards. They have different size, which is expected considering the orthogonal design where all line cards are directly plugged into the fabric cards.
Also, we have now two generations of fabric cards.
The v1 supports line cards equipped with Jericho and Jericho+ NPUs. The v2 are supporting the same line cards but also the new ones powered by Jericho2 ASICs.
Depending on the chassis size, each fabric card will be made of one or multiple Fabric Engine.

                     V1.       V2.
NCS5504.     1.         1.
NCS5508.     2.         2.
NCS5516.     6.         3.

Regardless of the generation, each router is using 6 fabric cards. They can operate with less than 6 but it could be at the expend of the bandwidth of some line cards. That's what we will detail in this blog post.

In the video below, we are showing a test with a 36x100G-SE line card used with 6 fabric cards then 5.

We are showing this and that and that.

Now, let's present the math used to identify the bandwidth available when losing one fabric card and how it affects each type of line cards in our portfolio.
