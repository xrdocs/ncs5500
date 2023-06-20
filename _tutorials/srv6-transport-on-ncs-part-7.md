---
published: true
date: '2023-05-19 22:39 +0530'
title: 'SRv6 on NCS 500/5500: QoS Propagation'
author: Paban Sarma
position: hidden
---
## Overview
In our previous tutorials in this series, we covered various aspects on SRv6 transport implementaion on the NCS 500/5500/5700 series platforms. In this tutorial, we will cover another important aspects, i.e. QoS propagation for SRv6 transport on the NCS 500 and NCS 5500 series platforms. The following figure shows a typical SRv6 encapsulated traffic and from the same it is evident that managing core quality of service for an SRv6 transport network is as simple as managing IPv6 QoS. The simplest way would be to manage the same, by use of the IPv6 DSCP or Precedence field in the SRv6 encapsulation field. 

<placeholder for different field in SRv6 packet>

## QoS Propagation Options

Coming to platform implemenation on the NCS 500/5500 and 5700 series routers, by default the QoS fields in the SRv6 header are not set. 

## QoS Propagation without ingress policy

## QoS Propagation with ingress Policy

## Example scenario with Traffic


## Conclusion
