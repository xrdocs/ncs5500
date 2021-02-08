---
published: false
date: '2021-02-03 10:16 +0530'
title: 'Y.1564 Service Activation Testing on NCS : Part2'
author: Paban Sarma
tags:
  - iosxr
  - NCS 5500
  - NCS 500
excerpt: >-
  Tutorial Explaining Y.1564 color aware mode configuration  and use case on NCS
  500 and NCS 5500 series routers
---
|Paban Sarma, Technical Marketing Engineer (pasarma@cisco.com) |  
|Chethan K .S, Software Test Lead (ches@cisco.com) |

## Overview

In [Previous Article](https://xrdocs.io/ncs5500/tutorials/y-1564-sadt-nc5x-part1/) we looked at the Y.1564 concepts and example configuration to run a Y.1564 Service Activation Test on NCS 5500 and NCS 500 routers. In this part, we will explore more capabilities such as color aware test and their application.

## Y.1564 Color Profile
An ethernet service activation test can be either color aware or color blind. By default, a test is color blind and only a single flow is generated at a rate configured in the information rate. The color aware mode is distinguished by use of a different COS marking. This different COS identifies the Excess Information Rate (EIR) flow. The committed flow uses the COS value provided in the “outer-COS” and “inner-COS” configuration. In a color aware flow, the EIR flow that is generated might have the discard eligibility indicator (DEI) set (dei =1). By default, it is not set (dei=0). The Committed information rate (CIR) needs to be configured for color-aware mode and EIR is obtained from the difference of information rate (IR) and committed information rate (CIR).

<div class="highlighter-rouge">
<pre class="highlight">
<code>
ethernet service-activation-test
 profile profile_#_1
  outer-cos 1
  <mark> color-aware <span style="color:blue"> cir 700 mbps</span> <span style="color:pink">  eir-color cos 0</span></mark>
  information-rate 1 gbps
  packet-size 512
</code>
</pre>
</div> 

The line `color-aware cir 700 mbps eir-color cos 0`  in the SADT profile denotes that this is a color aware profile. The committed information rate (CIR) is specified in the same configuration line. The excess information rate (EIR) is obtained from the difference of information rate (IR) and the CIR . When traffic is generated, traffic within CIR is marked with the outer-cos value in profile, traffic exceeding CIR profile is marked with the eir-color cos. 

There are certain restriction on cos values that can be used in an color aware Service Activation Profile shown in the below table:

|    <br>PCP value <br>   <br>(COS)    	|    <br>Support for Color Blind    	|    <br>Support for color aware w/o DEI (0)    	|    <br>Support for color aware  w/ DEI set (1)    	|
|--------------------------------------	|-----------------------------------	|-----------------------------------------------	|---------------------------------------------------	|
|    <br>0                             	|    <br>✔                          	|    <br>✔                                      	|    <br>✔                                          	|
|    <br>1                             	|    <br>✔                          	|    <br>✔                                      	|    <br>✔                                          	|
|    <br>2                             	|    <br>✔                          	|    <br>✔                                      	|    <br>X                                          	|
|    <br>3                             	|    <br>✔                          	|    <br>✔                                      	|    <br>X                                          	|
|    <br>4                             	|    <br>✔                          	|    <br>✔                                      	|    <br>X                                          	|
|    <br>5                             	|    <br>✔                          	|    <br>X                                      	|    <br>X                                          	|
|    <br>6                             	|    <br>✔                          	|    <br>✔                                      	|    <br>X                                          	|
|    <br>7                             	|    <br>✔                          	|    <br>X                                      	|    <br>X                                          	|


`Note: Color aware profile needs the cos values to be preserved in the packets. Therefore, color aware Service Activation Test  can’t be supported if we have pop operation configured on the attachment circuits (EFPs)`

## Y.1564 Color Aware Profile Configuration 

### Color aware profile without DEI bit

<div class="highlighter-rouge">
<pre class="highlight">
<code>
profile ca1
  outer-cos 4
  mode two-way
  duration 5 minutes
  color-aware cir 300 mbps eir-color cos 3
  description color_aware_sat_profile
  packet-size 1500
  information-rate 500 mbps
 !
</code>
</pre>
</div> 

The Profile Generates traffic at 500 Mbps and a CIR of 300 Mbps. The CIR traffic is marked with COS value of 4. The EIR is 200 Mbps (500-200) and EIR traffic is marked with COS value of 3. The following figures shows the snapshot of generated traffic.
![CIR flow generated by profile CA1]({{site.baseurl}}/images/CA1_CIR.png)

![EIR flow generated by profile CA1]({{site.baseurl}}/images/CA1_EIR.png)

### Color aware profile with DEI bit
<div class="highlighter-rouge">
<pre class="highlight">
<code>
profile ca2
  outer-cos 1
  mode two-way
  duration 5 minutes
  color-aware cir 300 mbps eir-color set-dei cos 0
  description color_aware_sat_profile_w_dei
  packet-size 1500
  information-rate 500 mbps
 !
</code>
</pre>
</div> 

This profile generates traffic at 500 Mbps with a CIR if 300 Mbps.  The CIR traffic is marked with COS value of 1. The EIR here is 200 Mbps (500- 300) and EIR traffic is marked with COS 0 and DEI bit for EIR traffic is also set to 1. The following figures shows the snapshot of generated traffic.
![CIR flow generated by profile CA2]({{site.baseurl}}/images/CA2_CIR.png)
![EIR flow generated by profile CA2]({{site.baseurl}}/images/CA2_EIR.png)


## Application: Validating Color Aware/Multi-CoS Bandwidth Profile

In this section, we will explore the application of  color aware traffic generation to validate a point-to-point L2 service with a multi COS bandwidth profile. The color aware profile uses two different COS values to generate a CIR and EIR flow. It can be used to validate a service where a color aware or a multi-COS policer is enforced 

### Scenario: 
A 500 Mbps point-to-point circuit needs to be provisioned with a multi COS bandwidth profile. Both the traffic classes are identified by COS value of 4 and 3. COS 4 has a committed rate of 300 Mbps and peak up to 500 Mbps, similarly COS 3 has a committed rate of 200 Mbps and peak  up to 500 Mbps. Total traffic is committed at 500 Mbps and peak rate is also 500 Mbps. This is achieved using a conform-aware hierarchical BWP where parent has a 500 Mbps CIR and PIR policer. The child classes have their individual policer with CIR of 300 and 200 Mbps respectively. Peak-rate for both child is 500 Mbps. The objective is to check if Committed rate is met for both child classes when total traffic rate matching both class cross 500 Mbps.

### Solution: 
Once the circuit and QoS all are in place, the service can be validated using a color aware   SADT profile. We can leverage the CIR and EIR flows to simultaneously check traffic from both class. We will generate same rate of traffic for both flows (1Gbps each) and check if cos4 is limited to 300 and cos3 is limited to 200 Mbps. Individual class peak-rate can be validated by simply using color blind profiles.

### Configuration and Verifications
![Topology used]({{site.baseurl}}/images/SADT_topo.png)

#### config on PE1

interface TenGigE0/0/0/0.1001 l2transport
 encapsulation dot1q 1001
 mtu 9000
 ethernet service-activation-test
  permit internal
 !
 service-policy input conform-parent
!

L2vpn
xconnect group vpws
  p2p ll_pw
   interface TenGigE0/0/0/0.1001
   neighbor ipv4 2.2.2.2 pw-id 1001
   
#### config on PE2

interface TenGigE0/0/0/2.1001 l2transport
 encapsulation dot1q 1001
 mtu 9000
 ethernet loopback
  permit internal
 !
  service-policy input conform-parent
!

L2vpn
xconnect group vpws
    p2p ll_pw
   interface TenGigE0/0/0/2.1001
   neighbor ipv4 1.1.1.1 pw-id 1001


#### Common QoS Config

policy-map conform-parent
 class class-default
  service-policy conform-child
  police rate 500 mbps peak-rate 500 mbps
  !
 !
 end-policy-map

class-map match-any green
 match COS 4
 end-class-map
!
class-map match-any yellow
 match COS 3
 end-class-map
policy-map conform-child
 class green
  police rate 300 mbps peak-rate 500 mbps
  !
 !
 class yellow
  police rate 200 mbps peak-rate 500 mbps
  !
 !
 class class-default
  police rate 0 bps
  !
 !
 end-policy-map


#### SADT Profile configuration:

ethernet service-activation-test
 profile color-aware
  outer-cos 4
  mode two-way
  duration 15 minutes
  color-aware cir 1 gbps eir-color cos 3
  packet-size 1500
  information-rate 2 gbps

ethernet service-activation-test
 profile color-blind
  outer-cos 4
  mode two-way
  duration 15 minutes
  packet-size 1500
  information-rate 2 gbps

The color aware profile used to validate the BWP is generating IR at 2 Gbps (1Gbps CIR & 2-1=1Gbps EIR). The expectation is we would see 70% loss for CIR and 80% loss for the EIR streams. To validate individual class can peak upto 500 Mbps, color blind profile is used at 2 Gbps and expected FLR is 75%.

### SADT Result

Now, we start the service activation test on the UNI interface. The loopback is enabled on the remote UNI in internal direction. From below results, we see that both during the test and after the test the cumulative traffic loss for class green (CIR, COS 4) is 70% which means 300 Mbps traffic is flowing through the circuit. For EIR it is 80% FLR meaning 200 Mbps is guaranteed.

ethernet  service-activation-test start interface TenGigE0/0/0/0.1001 profile color-aware destination 1.2.3 direction  internal




