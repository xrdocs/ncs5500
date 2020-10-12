---
published: false
date: '2020-10-12 15:11 +0200'
title: NCS-5500 Fabric Migration
author: Nicolas Fevrier
excerpt: v1 to v2 NCS-5500 ugprade procedure
position: hidden
---
{% include toc icon="table" title="NCS-5500 v1 to v2 Fabric Migration" %} 

You can find more content related to NCS-5500 and NCS-5700 following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Before insertion of 400GE line cards, the NCS-5500 needs to be modified to support packet forwarding and cooling. For instance, it's necessary to use "version 2" (v2) fabric cards and fan trays. During the first 3 years of its existence, the NCS-5500 chassis where shipped with v1 FC/FT.  
In this blog post, Benoit Mercier Des Rochettes ([Manager in CX organization](https://www.linkedin.com/in/benoit-mercier-des-rochettes-12960148/)) will detail the different step necessary to guarantee a smooth migration from v1 to v2.

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/XMQumuTkzmg?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

## High level description of the migration steps

- Lab preparation
- Configuration of the customer features on the router(s)
- Configuration of the scales on the generators
- Preparation of a realistic topology
- Perform software upgrade
- Verification of non regression on the existing features
- Documentation of the MOP

In this demo, we used an 8-slot chassis with NC55-36X100G-SE-S (powered by Jericho+ ASICs) and executing IOS XR 6.3.15.

Lab topology:

![test-topology.png]({{site.baseurl}}/images/test-topology.png){: .align-center}

The testbed is made of 3 routers (NCS-5508 with 36x100G-SE-S line card, NCS-5501-SE and ASR9000) and one traffic generator (TRex: [more details available here](https://trex-tgn.cisco.com/)).  
We will monitor 3 services (and flow) during the test:  
- L3VPN
- PW
- PW over RSVP-TE
A route generator completes this picture, advertising 996,940 IPv4 and 225,280 IPv6 routes over BGP (the internet routing scale projection for our customer).  

The purpose of this configuration being to identify if/when each step will be disruptive for the customer's services.

### Before getting started

With the following show command, we verify:
- line cards inserted
- the chassis type: 5508
- the Fan Trays type: NC55-5508-FAN is v1
- the Fabric Cards type: NC55-5508-FC is v1

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show platform
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/2/CPU0          NC55-36X100G-A-SE          IOS XR RUN        NSHUT
0/2/NPU0          Slice                      UP
0/2/NPU1          Slice                      UP
0/2/NPU2          Slice                      UP
0/2/NPU3          Slice                      UP
0/RP0/CPU0        NC55-RP-E(Active)          IOS XR RUN        NSHUT
0/RP1/CPU0        NC55-RP-E(Standby)         IOS XR RUN        NSHUT
0/FC0             NC55-5508-FC               OPERATIONAL       NSHUT
0/FC1             NC55-5508-FC               OPERATIONAL       NSHUT
0/FC2             NC55-5508-FC               OPERATIONAL       NSHUT
0/FC3             NC55-5508-FC               OPERATIONAL       NSHUT
0/FC4             NC55-5508-FC               OPERATIONAL       NSHUT
0/FC5             NC55-5508-FC               OPERATIONAL       NSHUT
0/FT0             NC55-5508-FAN              OPERATIONAL       NSHUT
0/FT1             NC55-5508-FAN              OPERATIONAL       NSHUT
0/FT2             NC55-5508-FAN              OPERATIONAL       NSHUT
0/SC0             NC55-SC                    OPERATIONAL       NSHUT
0/SC1             NC55-SC                    OPERATIONAL       NSHUT
RP/0/RP0/CPU0:5508#</code>
</pre>
</div>

We check the currently used IOS XR version

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show install active summary
    Active Packages: 18
        ncs5500-xr-6.3.15 version=6.3.15 [Boot image]
        ncs5500-mcast-2.1.0.0-r6315
        ncs5500-mpls-2.1.0.0-r6315
        ncs5500-mgbl-4.0.0.0-r6315
        ncs5500-mpls-te-rsvp-2.2.0.0-r6315
        ncs5500-isis-1.3.0.0-r6315
        ncs5500-k9sec-4.1.0.0-r6315
        ncs5500-common-pd-fib-1.1.0.1-r6315.CSCvi05806
        ncs5500-iosxr-fwding-5.0.0.4-r6315.CSCvi05806
        ncs5500-fwding-4.0.0.1-r6315.CSCvi25051
        ncs5500-dpa-fwding-4.1.0.7-r6315.CSCvh03531
        ncs5500-dpa-3.0.0.12-r6315.CSCvh03531
        cisco-klm-0.1.p1-r0.0.CSCvn55720.xr
        kernel-image-3.14.23-wr7.0.0.2-standard-3.14.p1-r0.1.CSCvn55720.xr
        cisco-klm-zermatt-0.1.p1-r0.0.CSCvn55720.xr
        ncs5500-os-support-4.0.0.3-r6315.CSCvq17485
        ncs5500-infra-4.1.0.11-r6315.CSCvj64412
        ncs5500-bgp-1.1.0.3-r6315.CSCvj90955
RP/0/RP0/CPU0:5508#show install committed summary
    Committed Packages: 28
        ncs5500-xr-6.3.15 version=6.3.15 [Boot image]
        ncs5500-mcast-2.1.0.0-r6315
        ncs5500-mpls-2.1.0.0-r6315
        ncs5500-mgbl-4.0.0.0-r6315
        ncs5500-mpls-te-rsvp-2.2.0.0-r6315
        ncs5500-isis-1.3.0.0-r6315
        ncs5500-k9sec-4.1.0.0-r6315
        ncs5500-infra-4.1.0.4-r6315.CSCvi54033
        ncs5500-common-pd-fib-1.1.0.1-r6315.CSCvi05806
        ncs5500-iosxr-fwding-5.0.0.4-r6315.CSCvi05806
        ncs5500-dpa-3.0.0.6-r6315.CSCvi05806
        ncs5500-dpa-fwding-4.1.0.4-r6315.CSCvi05806
        ncs5500-dpa-3.0.0.8-r6315.CSCvh75088
        ncs5500-infra-4.1.0.6-r6315.CSCvh03531
        ncs5500-os-support-4.0.0.1-r6315.CSCvi50726
        ncs5500-dpa-3.0.0.10-r6315.CSCvi25051
        ncs5500-fwding-4.0.0.1-r6315.CSCvi25051
        ncs5500-dpa-fwding-4.1.0.7-r6315.CSCvh03531
        ncs5500-dpa-3.0.0.12-r6315.CSCvh03531
        ncs5500-os-support-4.0.0.2-r6315.CSCvg80365
        cisco-klm-0.1.p1-r0.0.CSCvn55720.xr
        kernel-image-3.14.23-wr7.0.0.2-standard-3.14.p1-r0.1.CSCvn55720.xr
        cisco-klm-zermatt-0.1.p1-r0.0.CSCvn55720.xr
        ncs5500-infra-4.1.0.9-r6315.CSCvq17485
        ncs5500-os-support-4.0.0.3-r6315.CSCvq17485
        ncs5500-infra-4.1.0.11-r6315.CSCvj64412
        ncs5500-bgp-1.1.0.2-r6315.CSCvj15728
        ncs5500-bgp-1.1.0.3-r6315.CSCvj90955
RP/0/RP0/CPU0:5508#</code>
</pre>
</div>

It's important to verify with this show command output that both route processor processes are in sync. NSR status must be ready.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show redundancy summary
    Active Node    Standby Node
    -----------    ------------
     0/RP0/CPU0      0/RP1/CPU0 (Node Ready, NSR:Ready)
RP/0/RP0/CPU0:5508#</code>
</pre>
</div>

Verify we don't have any pending firmware upgrade. "CURRENT" is the expected status for all elements.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show hw-module fpd
                                                               FPD Versions
                                                               =================
Location   Card type        HWver FPD device       ATR Status   Running Programd
------------------------------------------------------------------------------
0/2       NC55-36X100G-A-SE 0.303 MIFPGA               CURRENT    0.03    0.03
0/2       NC55-36X100G-A-SE 0.303 Bootloader           CURRENT    0.13    0.13
0/2       NC55-36X100G-A-SE 0.303 DBFPGA               CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 IOFPGA               CURRENT    0.21    0.21
0/2       NC55-36X100G-A-SE 0.303 SATA                 CURRENT    5.00    5.00
0/RP0     NC55-RP           1.1   Bootloader           CURRENT    9.28    9.28
0/RP0     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/RP1     NC55-RP           1.1   Bootloader           CURRENT    9.28    9.28
0/RP1     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/FC0     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC0     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC1     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC1     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC2     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC2     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC3     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC3     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC4     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC4     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC5     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC5     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/SC0     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC0     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10
0/SC1     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC1     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10
RP/0/RP0/CPU0:5508#admin
sysadmin-vm:0_RP0# show hw-module fpd
                                                                   FPD Versions
                                                                ===============
Location  Card type         HWver FPD device       ATR Status   Run    Programd
-------------------------------------------------------------------------------
0/2       NC55-36X100G-A-SE 0.303 Bootloader           CURRENT    0.13    0.13
0/2       NC55-36X100G-A-SE 0.303 DBFPGA               CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 IOFPGA               CURRENT    0.21    0.21
0/2       NC55-36X100G-A-SE 0.303 SATA                 CURRENT    5.00    5.00
0/RP0     NC55-RP           1.1   Bootloader           CURRENT    9.28    9.28
0/RP0     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/RP1     NC55-RP           1.1   Bootloader           CURRENT    9.28    9.28
0/RP1     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/FC0     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC0     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC1     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC1     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC2     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC2     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC3     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC3     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC4     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC4     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/FC5     NC55-5508-FC      1.0   Bootloader           CURRENT    1.74    1.74
0/FC5     NC55-5508-FC      1.0   IOFPGA               CURRENT    0.16    0.16
0/SC0     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC0     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10
0/SC1     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC1     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10
sysadmin-vm:0_RP0#</code>
</pre>
</div>  

Make sure we have fpd auto-upgrade configured.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show running-config | i fpd
Building configuration...
fpd auto-upgrade enable
RP/0/RP0/CPU0:5508#admin

cisco connected from 127.0.0.1 using console on 5508
sysadmin-vm:0_RP0# show running-config | i fpd
fpd auto-upgrade enable
sysadmin-vm:0_RP0#</code>
</pre>
</div>  

We back up current configuration (and admin config) in both RP0 and RP1 harddisk.  
In normal condition, the system will reboot with a RP0 active and RP1 standby, but just in case RP0 is slow to restart and RP1 takes the ownership, it's better to have the configurations on both cards.  

<div class="highlighter-rouge">
<pre class="highlight">
<code>copy running-config harddisk:PRE_running_config_122719.txt
copy harddisk:PRE_running_config_122719.txt location 0/RP0/CPU0 harddisk: location 0/RP1/CPU0

admin
copy running-config harddisk:PRE_admin-running_config_122719.txt
copy harddisk:PRE_admin-running_config_122719.txt location 0/RP0 harddisk: location 0/RP1</code>
</pre>
</div>  

![Screen Shot 2020-10-12 at 4.30.58 PM.png]({{site.baseurl}}/images/Screen Shot 2020-10-12 at 4.30.58 PM.png){: .align-center}

IOS XR images can be downloaded on Software Download site:

![Screen Shot 2020-10-12 at 4.23.52 PM.png]({{site.baseurl}}/images/Screen Shot 2020-10-12 at 4.23.52 PM.png){: .align-center}

Extract the tar file and upload the iso and rpm files in a dedicated folder on harddisk:  
Then verify the integrity of the files with the following

<div class="highlighter-rouge">
<pre class="highlight">
<code>dir harddisk:/663
cd harddisk:/663
show md5 file ncs5500-isis-2.2.0.0-r663.x86_64.rpm
show md5 file ncs5500-mcast-3.1.0.0-r663.x86_64.rpm
show md5 file ncs5500-mini-x-6.6.3.iso
show md5 file ncs5500-mpls-2.1.0.0-r663.x86_64.rpm
show md5 file ncs5500-k9sec-3.1.0.0-r663.x86_64.rpm
show md5 file ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64.rpm
show md5 file ncs5500-mgbl-3.0.0.0-r663.x86_64.rpm</code>
</pre>
</div>  

After verifying the matching of the MD5 hashing, we can proceed to the installation.

<div class="highlighter-rouge">
<pre class="highlight">
<code>install add source /harddisk:/663 ncs5500-isis-2.2.0.0-r663.x86_64.rpm ncs5500-k9sec-3.1.0.0-r663.x86_64.rpm ncs5500-mcast-3.1.0.0-r663.x86_64.rpm ncs5500-mgbl-3.0.0.0-r663.x86_64.rpm ncs5500-mini-x-6.6.3.iso ncs5500-mpls-2.1.0.0-r663.x86_64.rpm ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64.rpm</code>
</pre>
</div>  



show install inactive summary


install prepare id 

install activate


-- POST-UPGRADE --


xr :
show hw-module fpd 
admin :
show hw-module fpd 

xr:
show install active summary
show install committed summary

admin :
show install active summary
show install committed summary
  
-----

## Configuration used for the test

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:NCS55K#show run
Building configuration...
!! IOS XR Configuration 6.6.3
!
hostname NCS55K
!
domain ipv4 host aslab.local 172.16.0.2
domain name aslab.local
domain name-server 172.16.0.2
telnet vrf mgmt ipv4 server max-servers 100
telnet vrf default ipv4 server max-servers 100

explicit-path name To-IOX
 index 10 next-address strict ipv4 unicast 75.75.127.3
 index 20 next-address strict ipv4 unicast 75.75.127.2
!
vrf mgmt
 address-family ipv4 unicast
 !
 address-family ipv6 unicast
 !
!
vrf EXTRATEST
 address-family ipv4 unicast
  import route-target
   2000:1000
  !
  export route-target
   2000:1000
  !
 !
!
fpd auto-upgrade enable
!
interface Bundle-Ether11
 description to ASR9006: 4x 100G
 mtu 9114
 ipv4 address 1.255.111.1 255.255.255.0
 ipv4 verify unicast source reachable-via any
 ipv6 verify unicast source reachable-via any
 ipv6 address 2001:1:255:111::1/64
 load-interval 30
 dampening
!
interface Loopback0
 ipv4 address 75.75.127.1 255.255.255.255
!
interface tunnel-te110
 ipv4 unnumbered Loopback0
 logging events lsp-status reoptimize
 logging events lsp-status state
 priority 3 3
 destination 75.75.127.2
 path-option 100 explicit name To-IOX
!
interface MgmtEth0/RP0/CPU0/0
 vrf mgmt
 ipv4 address 192.168.255.124 255.255.254.0
!
interface HundredGigE0/2/0/14
 description ASR9006 Hu0/1/0/2
 bundle id 11 mode active
 cdp
 lacp period short
 carrier-delay up 30 down 0
 load-interval 30
!
interface HundredGigE0/2/0/15
 description ASR9006 Hu0/1/0/3
 bundle id 11 mode active
 cdp
 lacp period short
 carrier-delay up 30 down 0
 load-interval 30
!
interface HundredGigE0/2/0/23
 mtu 9000
 load-interval 30
!
interface HundredGigE0/2/0/23.100 l2transport
 encapsulation dot1q 100
 rewrite ingress tag pop 1 symmetric
!
interface HundredGigE0/2/0/23.101 l2transport
 encapsulation dot1q 101
 rewrite ingress tag pop 1 symmetric
!
interface HundredGigE0/2/0/23.1000
 vrf EXTRATEST
 ipv4 address 11.0.0.1 255.255.255.0
 encapsulation dot1q 1000
!
interface HundredGigE0/2/0/30
 ipv4 address 1.250.1.2 255.255.255.252
 load-interval 30
!
interface preconfigure HundredGigE0/5/1/0
 description ASR9006 Hu0/1/0/0
 bundle id 11 mode active
 cdp
 lacp period short
 carrier-delay up 30 down 0
 load-interval 30
!
interface preconfigure HundredGigE0/5/1/1
 description ASR9006 Hu01/0/1
 bundle id 11 mode active
 cdp
 lacp period short
 carrier-delay up 30 down 0
 load-interval 30
!
router static
 vrf mgmt
  address-family ipv4 unicast
   0.0.0.0/0 172.16.0.1
  !
 !
!
router isis 1
 set-overload-bit on-startup 240
 is-type level-2-only
 net 49.0001.0000.0222.0032.00
 segment-routing global-block 200000 207999
 nsr
 nsf cisco
 log adjacency changes
 address-family ipv4 unicast
  metric-style wide
  mpls traffic-eng level-2-only
  mpls traffic-eng router-id 75.75.127.1
  segment-routing mpls sr-prefer
 !
 address-family ipv6 unicast
  metric-style wide
 !
 interface Bundle-Ether11
  bfd fast-detect ipv4
  bfd fast-detect ipv6
  point-to-point
  address-family ipv4 unicast
   fast-reroute per-prefix
   fast-reroute per-prefix ti-lfa
   metric 1
   mpls ldp sync
  !
  address-family ipv6 unicast
   metric 1
  !
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid index 1
  !
  address-family ipv6 unicast
  !
 !
 interface HundredGigE0/2/0/30
  point-to-point
  address-family ipv4 unicast
   fast-reroute per-prefix
   fast-reroute per-prefix ti-lfa
   metric 1
   adjacency-sid absolute 15001
  !
  address-family ipv6 unicast
   metric 1
  !
 !
!
snmp-server traps bgp cbgp2 updown
snmp-server traps bgp updown
router bgp 2000
 nsr
 bgp router-id 75.75.127.1
 bgp graceful-restart
 address-family ipv4 unicast
 !
 address-family ipv4 multicast
 !
 address-family vpnv4 unicast
 !
 neighbor 75.75.127.2
  remote-as 2000
  update-source Loopback0
  address-family ipv4 unicast
  !
  address-family vpnv4 unicast
  !
 !
 vrf EXTRATEST
  rd 1:1000
  address-family ipv4 unicast
   redistribute connected
  !
 !
!
l2vpn
 router-id 75.75.127.1
 load-balancing flow src-dst-ip
 snmp mib pseudowire statistics
 pw-class PORT_PW
  encapsulation mpls
   protocol ldp
   control-word
   transport-mode ethernet
   load-balancing
    flow-label both
   !
  !
 !
 pw-class strictpath-to-iox
  encapsulation mpls
   protocol ldp
   control-word
   transport-mode ethernet
   preferred-path interface tunnel-te 110 fallback disable
  !
 !
 xconnect group g2
  p2p 100
   interface HundredGigE0/2/0/23.100
   neighbor ipv4 75.75.127.2 pw-id 100
    pw-class PORT_PW
   !
  !
  p2p 101
   interface HundredGigE0/2/0/23.101
   neighbor ipv4 75.75.127.2 pw-id 101
    pw-class strictpath-to-iox
   !
  !
 !
!
mpls oam
!
rsvp
 interface Bundle-Ether11
 !
 interface HundredGigE0/2/0/30
 !
!
mpls traffic-eng
 interface Bundle-Ether11
  admin-weight 6
 !
 interface HundredGigE0/2/0/30
  admin-weight 20
 !
!
mpls ldp
 log
  neighbor
  nsr
 !
 nsr
 session backoff 5 10
 router-id 75.75.127.1
 !
 interface Bundle-Ether11
 !
 interface HundredGigE0/2/0/30
 !
!
snmp-server traps pim neighbor-change
snmp-server traps pim invalid-message-received
snmp-server traps pim rp-mapping-change
snmp-server traps pim interface-state-change
!
segment-routing
 local-block 15000 15999
 traffic-eng
  logging
   policy status
  !
  policy POLICY1
   color 20 end-point ipv4 75.75.127.2
   candidate-paths
    preference 50
     explicit segment-list SIDLIST1
     !
    !
   !
  !
 !
!
xml agent tty
!
xml agent
!
snmp-server traps sensor
snmp-server traps fru-ctrl
netconf agent tty
!
netconf-yang agent
 ssh
!
lldp
!
ssh client source-interface MgmtEth0/RP0/CPU0/0
ssh server rate-limit 40
ssh server session-limit 30
ssh server v2
ssh server vrf mgmt
ssh server vrf default
ssh server netconf vrf default
snmp-server traps fabric bundle link
snmp-server traps fabric bundle state
end

RP/0/RP0/CPU0:NCS55K#</code>
</pre>
</div>


