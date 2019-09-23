---
published: false
date: '2019-09-23 17:22 +0200'
title: 'Testing NDR on NCS5500 36x 100GE Line Cards [Lab Series 02]'
author: Nicolas Fevrier
excerpt: Test to measure and explain the NDR on NCS5500 36x100G-SE line cards
position: hidden
tags:
  - ncs5500
  - lab series
  - testing
  - ndr
---
{% include toc icon="table" title="Testing NDR on 36x100G-SE" %}

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

This test is the second episode of our new Lab Series.  
You can find a detailed explanation on the purpose and also a link to all other tests in this xrdocs.io post:  [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)  
Last week, we run multiple tests in a very large test bed. Among the topics covered, we measured the NDR with a very long snake.  
The concept of Non Drop Rate deserves dedicated explanations.  
In this article and video, we will explain what it represents. We will demonstrate why the snake topology is not the best to reach the full capability of the ASIC and what happens when you push the system to its limit.

## Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/_i7xahJO1Yg?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

## What is NDR?

## Test results

### Under 130B/pkt

### Between 230B/pkt and 278B/pkt

### Performance per 100G ports

Test with 9 ports, asymmetrical port allocation

Test with just 2 ports at 64B and 256B

## Conclusion

