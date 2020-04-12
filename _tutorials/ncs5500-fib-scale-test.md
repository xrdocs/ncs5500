---
published: false
date: '2020-04-13 00:05 +0200'
title: 'NCS5500 FIB Scale Test [Lab Series 04] '
author: Nicolas Fevrier
excerpt: Testing full internet views and up to 4M prefixes
position: hidden
---
{% include toc icon="table" title="NCS5500 FIB Scale Tests" %} 

![LabSeries-banner-thin.png]({{site.baseurl}}/images/LabSeries-banner-thin.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Episode 04 of the lab series, today, we will talk about NCS5500 FIB. More specifically, we will verify the number of routes we can store in systems with Jericho+ and eTCAM.  
The goal of these blog posts and videos is to describe tests performed in lab, detail the methodology and the results.  
They are extracted from customers POC (proof of concept). With this information, we hope it will speed up your validation process and provide additional information on the NCS5500 platforms internals.  
All former tests are listed here: [https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/](https://xrdocs.io/ncs5500/tutorials/ncs5500-lab-series/)

## Video

In the video below, we are showing a test with a 36x100G-SE line card used with 6 fabric cards then 5.

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/oglYEDpKsLY?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

