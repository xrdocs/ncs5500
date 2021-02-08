---
published: true
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

<div class="highlighter-rouge">
<pre class="highlight">
<code>
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
</code>
</pre>
</div> 

#### config on PE2

<div class="highlighter-rouge">
<pre class="highlight">
<code>
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
</code>
</pre>
</div> 

#### Common QoS Config

<div class="highlighter-rouge">
<pre class="highlight">
<code>
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
</code>
</pre>
</div> 

#### SADT Profile configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>
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
</code>
</pre>
</div> 

The color aware profile used to validate the BWP is generating IR at 2 Gbps (1Gbps CIR & 2-1=1Gbps EIR). The expectation is we would see 70% loss for CIR and 80% loss for the EIR streams. To validate individual class can peak upto 500 Mbps, color blind profile is used at 2 Gbps and expected FLR is 75%.

### SADT Result

Now, we start the service activation test on the UNI interface. The loopback is enabled on the remote UNI in internal direction. From below results, we see that both during the test and after the test the cumulative traffic loss for class green (CIR, COS 4) is 70% which means 300 Mbps traffic is flowing through the circuit. For EIR it is 80% FLR meaning 200 Mbps is guaranteed.

`ethernet  service-activation-test start interface TenGigE0/0/0/0.1001 profile color-aware destination 1.2.3 direction  internal`

`Note: The remote PE must enable a MAC swap loop for correct SADT statistics. EDPL functionality can be used on Cisco NCS devices for this.`



<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:PE1# RP/0/RP0/CPU0:T-2006#show ethernet service-activation-test  interface TenGigE0/0/0/0.1001

Mon Feb  1 10:50:24.335 UTC
Interface TenGigE0/0/0/0.1001
  Service activation tests permitted (internal only)
  Test completed:
    Duration 15 minute(s)
    Information rate 2 Gbps
    <mark>Color-aware, CIR: 1 Gbps, EIR: CoS 3</mark>
    Internal, Two-way, Destination 00:01:00:02:00:03
    Packet size 1500, Pattern hex 0x00
    <mark>Outer CoS 4</mark>

  Results:
    Step 1, Information Rate 2 Gbps
      <mark>CIR packets:</mark>
        Tx packets: 74923274, bytes: 112384911000
        Rx packets: 22180963, bytes: 33271444500
        FL: 52742311, <mark>FLR: 70%</mark>
        FD: Min 11.736us, Mean 14.605us, Max 17.532us
        IFDV: Not supported
        Out of order packets: 11220557 (15%)
        Error packets: 0 (0%)

      <mark>EIR packets:</mark>
        Tx packets: 74923274, bytes: 112384911000
        Rx packets: 14696226, bytes: 22044339000
        FL: 60227048, <mark>FLR: 80%</mark>
        FD: Min 12.224us, Mean 14.625us, Max 17.480us
        IFDV: Not supported
        Out of order packets: 10596475 (14%)
        Error packets: 0 (0%)

RP/0/RP0/CPU0:T-2006#show  policy-map  int TenGigE 0/0/0/0.1001 input 
Mon Feb  1 10:20:32.468 UTC

TenGigE0/0/0/0.1001 input: conform-parent

Class class-default
  Classification statistics          (packets/bytes)     (rate - kbps)
    Matched             :           191993780/288374657560         1999997
    <mark>Transmitted         :            47299928/71044491856          492736</mark>
    Total Dropped       :           144693852/217330165704         1507261
  Policing statistics                (packets/bytes)     (rate - kbps) 
    Policed(conform)    :            47299928/71044491856          492763
    Policed(exceed)     :                   0/0                    0
    Policed(violate)    :           144693852/217330165704         1507327
    Policed and dropped :           144693852/217330165704       

  Policy conform-child Class green
    Classification statistics          (packets/bytes)     (rate - kbps)
      Matched             :            21072224/31650480448          999951
      Transmitted         :             6230480/9358180960           295643
      Total Dropped       :            14841744/22292299488          704308
    Policing statistics                (packets/bytes)     (rate - kbps) 
      <mark>Policed(conform)    :             6230480/9358180960           295643</mark>
      Policed(exceed)     :                   0/0                    0
      Policed(violate)    :            14841744/22292299488          704308
      Policed and dropped :            14841744/22292299488        
      Policed and dropped(parent policer)  : 0/0
          
  Policy conform-child Class yellow
    Classification statistics          (packets/bytes)     (rate - kbps)
      Matched             :            95996887/144187324274         1000046
      <mark>Transmitted         :            41069448/61686310896          197093</mark>
      Total Dropped       :            54927439/82501013378          802953
    Policing statistics                (packets/bytes)     (rate - kbps) 
      Policed(conform)    :            41069448/61686310896          197093
      Policed(exceed)     :                   0/0                    0
      Policed(violate)    :            54927439/82501013378          802953
      Policed and dropped :            54927439/82501013378        
      Policed and dropped(parent policer)  : 0/0

</code>
</pre>
</div>

As we see from above results Committed rate for both service is met. The FLR of ~70% and 80% for the CIR and EIR flows mean a throughput of 300 Mbps and 200 Mbps for the CoS 4 and CoS 3 respectively.   
The below output for the color blind profile can validate that individually, the CoS 4  class can reach a peak-rate of 500 Mbps. The test is run on ideal scenario and the FLR of 75% on 2 Gbps IR indicates that a throughput of 500 Mbps is achieved.

In Both of the test result there is out of order packets which is expected as OOO is flagged if a recieved packet sequence is different from the expected sequence and it is bound to happen when some of the generated packets are loss due to QoS enforecment. 

<div class="highlighter-rouge">
<pre class="highlight">
<code>
RP/0/RP0/CPU0:T-2006#show  ethernet service-activation-test interface TenGigE0/0/0/0.1001
Mon Feb  1 16:43:44.589 UTC
Interface TenGigE0/0/0/0.1001
  Service activation tests permitted (internal only)
  Test completed:
    Duration 15 minute(s)
    Information rate 2 Gbps
    <mark>Color-blind</mark>
    Internal, Two-way, Destination 00:01:00:02:00:03
    Packet size 1500, Pattern hex 0x00
    <mark>Outer CoS 4</mark>

  Results:
    Step 1, Information Rate 2 Gbps
      <mark>CIR packets:</mark>
        Tx packets: 149851022, bytes: 224776533000
        Rx packets: 36768654, bytes: 55152981000
        FL: 113082368, <mark>FLR: 75%</mark>
        FD: Min 12.188us, Mean 14.825us, Max 20.252us
        IFDV: Not supported
        Out of order packets: 11284818 (8%)
        Error packets: 0 (0%)

      EIR packets:
        Tx packets: 0, bytes: 0
        Rx packets: 0, bytes: 0
        FL: 0, FLR: 0%
        FD: Min 0.000us, Mean 0.000us, Max 0.000us
        IFDV: Min 0.000us, Mean 0.000us, Max 0.000us
        Out of order packets: 0 (0%)
        Error packets: 0 (0%)

RP/0/RP0/CPU0:T-2006#show policy-map interface tenGigE 0/0/0/0.1001
Mon Feb  1 16:50:17.880 UTC

TenGigE0/0/0/0.1001 input: conform-parent

Class class-default
  Classification statistics          (packets/bytes)     (rate - kbps)
    Matched             :            45190770/67876536540          2000402
    <mark>Transmitted         :            11133061/16721857622          492788</mark>
    Total Dropped       :            34057709/51154678918          1507614
  Policing statistics                (packets/bytes)     (rate - kbps) 
    Policed(conform)    :            11133061/16721857622          492788
    Policed(exceed)     :                   0/0                    0
    Policed(violate)    :            34057709/51154678918          1507614
    Policed and dropped :            34057709/51154678918        

  Policy conform-child Class green
    Classification statistics          (packets/bytes)     (rate - kbps)
      Matched             :            45190770/67876536540          2000402
      <mark>Transmitted         :            11133061/16721857622          492788</mark>
      Total Dropped       :            34057709/51154678918          1507614
    Policing statistics                (packets/bytes)     (rate - kbps)
      Policed(conform)    :            11133061/16721857622          492788
      Policed(exceed)     :                   0/0                    0
      Policed(violate)    :            34057709/51154678918          1507614
      Policed and dropped :            34057709/51154678918
      Policed and dropped(parent policer)  : 0/0

  Policy conform-child Class yellow
    Classification statistics          (packets/bytes)     (rate - kbps)
      Matched             :                   0/0                    0
      Transmitted         :                   0/0                    0
      Total Dropped       :                   0/0                    0
    Policing statistics                (packets/bytes)     (rate - kbps)
      Policed(conform)    :                   0/0                    0
      Policed(exceed)     :                   0/0                    0
      Policed(violate)    :                   0/0                    0
      Policed and dropped :                   0/0
      Policed and dropped(parent policer)  : 0/0

  Policy conform-child Class class-default
    Classification statistics          (packets/bytes)     (rate - kbps)
      Matched             :                   0/0                    0
      Transmitted         :                   0/0                    0
      Total Dropped       :                   0/0                    0
    Policing statistics                (packets/bytes)     (rate - kbps)
      Policed(conform)    :                   0/0                    0
      Policed(exceed)     :                   0/0                    0
      Policed(violate)    :                   0/0                    0
      Policed and dropped :                   0/0
      Policed and dropped(parent policer)  : 0/0
Policy Bag Stats time: 1612198187756  [Local Time: 02/01/21 16:49:47.756]
</code>
</pre>
</div>

## Conclusion
We covered Y.1564 color aware test profile and various aspects of the configuration. We have also illustrated one use case scenario, where color aware SADT comes handy in verifying a L2 service with multi-CoS/color aware bandwidth profile.
