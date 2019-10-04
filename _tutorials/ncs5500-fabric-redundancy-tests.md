---
published: true
date: '2019-10-03 08:52 +0200'
title: NCS5500 Fabric Redundancy Tests
author: Nicolas Fevrier
excerpt: How an NCS5500 modular chassis behaves with 5 out of 6 fabric cards?
tags:
  - ncs5500
  - fabric
  - redundancy
  - lab
  - test
position: hidden
---
{% include toc icon="table" title="NCS5500 Fabric Redundancy Tests" %} 

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Third episode of the lab series, today, we will talk about NCS5500 fabric. More specifically, we will try to qualify the impact of losing a fabric card in a chassis.  
The goal of these blog posts is to describe tests performed in lab, detail the methodology and the results, and finally provide additional information on the internals of the NCS5500 platforms.  
All the former tests are listed here: [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)

## Video

In the video below, we are showing a test with a 36x100G-SE line card used with 6 fabric cards then 5.

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/SZHuq7d2QOM?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## Architecture

As you certainly know, the NCS5500 exists in fixed systems and modular chassis. For this second category, we have 4-slot, 8-slot and 16-slot versions.  

![3-chassis.png]({{site.baseurl}}/images/3-chassis.png){: .align-center}


If they can all mix and match different type of line cards, each chassis type is using specific fabric cards. They have different size, which is expected considering the orthogonal design where all line cards are directly plugged into the fabric cards.

![archi.png]({{site.baseurl}}/images/archi.png){: .align-center}

Also, we have now two generations of fabric cards.  
The v1 supports line cards equipped with Jericho and Jericho+ NPUs. The v2 are supporting the same line cards but also the new ones powered by Jericho2 ASICs.  

Depending on the chassis size, each fabric card will be made of one or multiple Fabric Engine:  

In the first generation fabric cards, we have one or several FE3600 ASICs:

![SFE-v1.png]({{site.baseurl}}/images/SFE-v1.png){: .align-center}

In the second generation, we will use a new fabric engine named "Ramon" (or FE9600)

![SFE-v2.png]({{site.baseurl}}/images/SFE-v2.png){: .align-center}

In summary:

|  | v1 | v2 |
|:-----:|:-----:|:-----:|
| NCS5504 | 1 | 1 |
| NCS5508 | 2 | 2 |
| NCS5516 | 6 | 3 |

Regardless of the generation, each router is using 6 fabric cards. They can operate with less than 6 but it could be at the expend of the bandwidth of some line cards. That's what we will detail in this blog post.

## Theory

Now, let's present the math used to identify the bandwidth available when losing one fabric card and how it affects each type of line cards in our portfolio.

## Test results


