---
published: true
date: '2022-09-23 10:06 +0530'
title: Implementing Layer2 VPN Over SRv6 Transport on NCS 5500/500
position: hidden
author: Paban Sarma
excerpt: >-
  L2VPN config and verification over SRV6 Transport on NCS 5500/500 series
  platforms
---
{% include toc icon="table" title="Table of Contents" %}

|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com)|  
|Tejas Lad, Technical Marketing Engineer (telad@cisco.com)|

## Overview

Until now, we covered [SRv6 Transport](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-1/) and [Layer3 VPN](https://xrdocs.io/ncs5500/tutorials/srv6-transport-on-ncs-part-2/) over SRv6 transport on NCS 5500 and NCS 500 platforms. In this tutorial, we will cover the impelementaion of EVPN based point-to-point l2vpn service (EVPN-VPWS) over SRv6. 