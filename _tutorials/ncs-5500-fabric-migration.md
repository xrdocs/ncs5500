---
published: true
date: '2020-10-12 15:11 +0200'
title: NCS-5500 Fabric Migration
author: Nicolas Fevrier
excerpt: v1 to v2 NCS-5500 ugprade procedure
position: top
---
{% include toc icon="table" title="NCS-5500 v1 to v2 Fabric Migration" %} 

You can find more content related to NCS-5500 and NCS-5700 following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

Before insertion of 400GE line cards, the NCS-5500 needs to be modified to support packet forwarding and cooling. For instance, it's necessary to use "version 2" (v2) fabric cards and fan trays. During the first 3 years of its existence, the NCS-5500 chassis where shipped with v1 FC/FT.  
In this blog post, Benoit Mercier Des Rochettes ([Manager in CX organization](https://www.linkedin.com/in/benoit-mercier-des-rochettes-12960148/)) will detail the different step necessary to guarantee a smooth migration from v1 to v2.  
At the moment of this video and blog publication, the v2 "commons" were only available for 8-slot and 16-slot chassis. The 4-slot version being in the roadmap.

Note: this is the short version of the MOP prepared by the CX team for this migration. The exhaustive one being specific to customer's requirement, we purposefully removed a lot of content for this article.
{: .notice--info}

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/XMQumuTkzmg?autoplay=1" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>{: .align-center}

## Lab topology

![test-topology.png]({{site.baseurl}}/images/test-topology.png){: .align-center}

The testbed is made of 3 routers (NCS-5508 with 36x100G-SE-S line card, NCS-5501-SE and ASR9000) and one traffic generator (TRex: [more details available here](https://trex-tgn.cisco.com/)).  
We will monitor 3 services (and flows) during the test:  
- L3VPN
- PW
- PW over RSVP-TE

A route generator completes this picture, advertising 996,940 IPv4 and 225,280 IPv6 routes over BGP (the internet routing scale projection for our customer).  

The purpose of this configuration being to identify if/when each step will be disruptive for the customer's services.

## Migration steps

In this demo, we used an 8-slot chassis with NC55-36X100G-SE-S (powered by Jericho+ ASICs) and starting with IOS XR 6.3.15.

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

It's important to verify with this show command output that both route processor processes are in sync. NSR status must be "Ready".

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

### Uploading IOS XR images and verification

![Screen Shot 2020-10-12 at 4.30.58 PM.png]({{site.baseurl}}/images/Screen Shot 2020-10-12 at 4.30.58 PM.png){: .align-center}

IOS XR images can be downloaded on Software Download site:

![Screen Shot 2020-10-12 at 4.23.52 PM.png]({{site.baseurl}}/images/Screen Shot 2020-10-12 at 4.23.52 PM.png){: .align-center}

Extract the tar file and upload the iso and rpm files in a dedicated folder on harddisk:  
Then verify the integrity of the files with the following

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#cd harddisk:/663
RP/0/RP0/CPU0:5508#show md5 file ncs5500-isis-2.2.0.0-r663.x86_64.rpm
f48a8fc9bdf03848a8e1a9e02d30a9cd
RP/0/RP0/CPU0:5508#show md5 file ncs5500-mcast-3.1.0.0-r663.x86_64.rpm
cd587e15d686eec15b75d64f1c08e482
RP/0/RP0/CPU0:5508#show md5 file ncs5500-mini-x-6.6.3.iso
418e65ff1228b17ec376c768ba305abe
RP/0/RP0/CPU0:5508#show md5 file ncs5500-mpls-2.1.0.0-r663.x86_64.rpm
4d58ba5c05a677b3e58c65d528344270
RP/0/RP0/CPU0:5508#show md5 file ncs5500-k9sec-3.1.0.0-r663.x86_64.rpm
c9ea261c1416b529b89b2691a5a22e9f
RP/0/RP0/CPU0:5508#show md5 file ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64$
65799fa3c474ae149bec05c73abfa93d
RP/0/RP0/CPU0:5508#show md5 file ncs5500-mgbl-3.0.0.0-r663.x86_64.rpm
71049785d117e75545d574a1e874a629
RP/0/RP0/CPU0:5508#</code>
</pre>
</div>  

### Software installation

After verifying the matching of the MD5 hashing, we can proceed to the installation.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#install add source /harddisk:/663 ncs5500-isis-2.2.0.0-r663.x86_64.rpm ncs5500-k9sec-3.1.0.0-r663.x86_64.rpm ncs5500-mcast-3.1.0.0-r663.x86_64.rpm ncs5500-mgbl-3.0.0.0-r663.x86_64.rpm ncs5500-mini-x-6.6.3.iso ncs5500-mpls-2.1.0.0-r663.x86_64.rpm ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64.rpm
 Install operation 6 started by cisco:
 install add source /harddisk:/663 ncs5500-isis-2.2.0.0-r663.x86_64.rpm ncs5500-k9sec-3.1.0.0-r663.x86_64.rpm ncs5500-mcast-3.1.0.0-r663.x86_64.rpm ncs5500-mgbl-3.0.0.0-r663.x86_64.rpm ncs5500-mini-x-6.6.3.iso ncs5500-mpls-2.1.0.0-r663.x86_64.rpm ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64.rpm
Dec 19 15:55:19 Install operation will continue in the background
RP/0/RP0/CPU0:5508#show install request
The install add operation 6 is 60% complete
RP/0/RP0/CPU0:5508#show install request
The install add operation 6 is 60% complete
RP/0/RP0/CPU0:Dec 19 16:08:50.052 : sdr_instmgr[1188]: %INSTALL-INSTMGR-2-OPERATION_SUCCESS : Install operation 6 finished successfully           
Dec 19 16:08:51 Install operation 6 finished successfully
RP/0/RP0/CPU0:5508#show install inactive summary
10 inactive package(s) found:
    ncs5500-mpls-te-rsvp-4.1.0.0-r663
    cisco-klm-zermatt-0.1-r0.0.xr
    ncs5500-isis-2.2.0.0-r663
    ncs5500-mcast-3.1.0.0-r663
    ncs5500-mini-x-6.6.3
    cisco-klm-0.1-r0.0.xr
    ncs5500-k9sec-3.1.0.0-r663
    ncs5500-mpls-2.1.0.0-r663
    kernel-image-3.14.23-wr7.0.0.2-standard-3.14-r0.1.xr
    ncs5500-mgbl-3.0.0.0-r663
RP/0/RP0/CPU0:5508# </code>
</pre>
</div>  

Then the preparation:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#install prepare id 4
Nov 18 08:16:07 Install operation 5 started by cisco:
  install prepare id 4
Nov 18 08:16:07 Package list:
Nov 18 08:16:08     ncs5500-mpls-2.1.0.0-r663.x86_64
Nov 18 08:16:08     ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64
Nov 18 08:16:08     ncs5500-isis-2.2.0.0-r663.x86_64
Nov 18 08:16:08     ncs5500-mgbl-3.0.0.0-r663.x86_64
Nov 18 08:16:08     ncs5500-mini-x-6.6.3
Nov 18 08:16:08     ncs5500-mcast-3.1.0.0-r663.x86_64
Nov 18 08:16:08     ncs5500-k9sec-3.1.0.0-r663.x86_64
Nov 18 08:16:14 Install operation will continue in the background
RP/0/RP0/CPU0:5508#
RP/0/RP0/CPU0:Nov 18 08:16:43.510 UTC: sdr_instmgr[1188]: %PKT_INFRA-FM-6-FAULT_INFO : INSTALL-IN-PROGRESS :DECLARE :0/RP0/CPU0: INSTALL_IN_PROGRESS Alarm : being DECLARED for the system
RP/0/RP0/CPU0:Nov 18 08:17:20.002 UTC: resmon[175]: %HA-HA_WD-4-DISK_WARN : A monitored device / ( rootfs:/ ) is above 80% utilization. Current utilization = 83. Please remove unwanted user files and configuration rollback points.
RP/0/RP0/CPU0:5508#show install request
The install prepare operation 5 is 40% complete
RP/0/RP0/CPU0:5508#
RP/0/RP0/CPU0:Nov 18 08:32:19.970 UTC: resmon[175]: %HA-HA_WD-3-DISK_ALARM_ALERT : A monitored device / ( rootfs:/ ) is above 80% utilization. Current utilization = 83. Please remove unwanted user files and configuration rollback points.
RP/0/RP0/CPU0:Nov 18 08:33:01.888 UTC: sdr_instmgr[1188]: %INSTALL-INSTMGR-2-OPERATION_SUCCESS : Install operation 5 finished successfully
Nov 18 08:33:04 Install operation 5 finished successfully
RP/0/RP0/CPU0:Nov 18 08:33:16.498 UTC: SSHD_[69004]: %SECURITY-SSHD-6-INFO_REKEY : Server initiated time rekey for session 0 , session_rekey_count = 1
RP/0/RP0/CPU0:Nov 18 08:33:19.971 UTC: resmon[175]: %HA-HA_WD-6-DISK_NORMAL : Device / ( rootfs:/ ) usage 43.

RP/0/RP0/CPU0:5508#</code>
</pre>
</div>  

And finally the activation:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#install activate id 6
Dec 20 09:37:31 Install operation 9 started by cisco:
  install activate id 6
Dec 20 09:37:31 Package list:
Dec 20 09:37:31     ncs5500-is	is-2.2.0.0-r663.x86_64
Dec 20 09:37:31     ncs5500-k9sec-3.1.0.0-r663.x86_64
Dec 20 09:37:31     ncs5500-mcast-3.1.0.0-r663.x86_64
Dec 20 09:37:31     ncs5500-mgbl-3.0.0.0-r663.x86_64
Dec 20 09:37:31     ncs5500-mini-x-6.6.3
Dec 20 09:37:31     ncs5500-mpls-2.1.0.0-r663.x86_64
Dec 20 09:37:31     ncs5500-mpls-te-rsvp-4.1.0.0-r663.x86_64
This install operation will reload the system, continue?
 [yes/no]:[yes] yes

RP/0/RP1/CPU0:Nov 14 14:03:30.464 UTC: fpd-serv[193]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :FPD-NEED-UPGRADE :DECLARE :0/2:
RP/0/RP1/CPU0:Nov 14 14:03:30.959 UTC: fpd-serv[193]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :FPD-NEED-UPGRADE :DECLARE :0/RP0:
RP/0/RP1/CPU0:Nov 14 14:03:31.597 UTC: fpd-serv[193]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :FPD-NEED-UPGRADE :DECLARE :0/RP1:
0/2/ADMIN0:Nov 14 14:03:35.485 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 11 percent for fpd Bootloader@location 0/2.
0/2/ADMIN0:Nov 14 14:03:40.486 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 12 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:03:40.983 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 1 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:03:41.612 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 1 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:03:45.486 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 13 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:03:45.983 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 2 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:03:46.612 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 2 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:03:50.487 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 15 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:03:50.984 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 3 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:03:51.613 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 4 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:03:55.487 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 16 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:03:55.984 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 4 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:03:56.613 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 5 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:00.487 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 18 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:00.984 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 5 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:01.613 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 6 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:05.488 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 19 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:05.984 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 7 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:06.613 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 7 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:10.488 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 22 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:10.985 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 8 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:11.614 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 8 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:15.488 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 28 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:15.985 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 9 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:16.614 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 9 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:20.489 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 34 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:20.985 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 10 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:21.614 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 10 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:25.489 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 39 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:25.986 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 11 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:26.615 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 12 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:30.490 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 44 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:30.986 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 12 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:31.615 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 13 percent for fpd Bootloader@location 0/RP1.
0/RP0/ADMIN0:Nov 14 14:04:35.986 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 13 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:36.615 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 14 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:38.490 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 52 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:40.987 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 14 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:41.615 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 15 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:43.490 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 56 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:45.987 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 15 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:46.616 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 16 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:48.490 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 61 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:50.987 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 16 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:51.616 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 17 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:53.491 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 65 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:04:55.988 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 17 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:04:56.616 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 18 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:04:58.491 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 71 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:00.988 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 18 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:01.616 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 19 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:03.491 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 75 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:05.988 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 19 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:06.617 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 27 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:10.492 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 81 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:10.988 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 25 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:11.617 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 36 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:15.492 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 83 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:15.989 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 33 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:16.617 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 44 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:20.492 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 86 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:20.989 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 41 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:23.617 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 56 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:25.493 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 88 percent for fpd Bootloader@location 0/2.
0/RP1/ADMIN0:Nov 14 14:05:28.618 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 61 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:30.493 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 91 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:30.989 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 55 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:33.618 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 68 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:35.493 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 93 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:35.990 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 61 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:38.618 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 74 percent for fpd Bootloader@location 0/RP1.
0/2/ADMIN0:Nov 14 14:05:40.494 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 96 percent for fpd Bootloader@location 0/2.
0/RP0/ADMIN0:Nov 14 14:05:40.990 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 67 percent for fpd Bootloader@location 0/RP0.
0/2/ADMIN0:Nov 14 14:05:45.494 UTC: card_mgr[2507]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 98 percent for fpd Bootloader@location 0/2.
0/2/ADMIN0:Nov 14 14:05:45.961 UTC: card_mgr[2507]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader@0/2 image programming completed with UPGRADE DONE state Info: [Programming Done]
0/2/ADMIN0:Nov 14 14:05:45.961 UTC: card_mgr[2507]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader @location 0/2 upgrade completed.
0/RP1/ADMIN0:Nov 14 14:05:45.962 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_SW_OPERATIONAL : Card: 0/2 software state going to Operational
0/RP1/ADMIN0:Nov 14 14:05:45.962 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_HW_OPERATIONAL : Card: 0/2 hardware state going to Operational
0/RP0/ADMIN0:Nov 14 14:05:45.990 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 73 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:46.618 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 84 percent for fpd Bootloader@location 0/RP1.
0/RP1/ADMIN0:Nov 14 14:05:51.619 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 89 percent for fpd Bootloader@location 0/RP1.
0/RP0/ADMIN0:Nov 14 14:05:53.991 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 83 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:05:56.619 UTC: card_mgr[2613]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 94 percent for fpd Bootloader@location 0/RP1.
0/RP0/ADMIN0:Nov 14 14:05:58.991 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 88 percent for fpd Bootloader@location 0/RP0.
0/RP1/ADMIN0:Nov 14 14:06:00.644 UTC: card_mgr[2613]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader@0/RP1 image programming completed with UPGRADE DONE state Info: [Programming Done]
0/RP1/ADMIN0:Nov 14 14:06:00.645 UTC: card_mgr[2613]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader @location 0/RP1 upgrade completed.
0/RP1/ADMIN0:Nov 14 14:06:00.648 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_SW_OPERATIONAL : Card: 0/RP1 software state going to Operational
0/RP1/ADMIN0:Nov 14 14:06:00.649 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_HW_OPERATIONAL : Card: 0/RP1 hardware state going to Operational
0/RP0/ADMIN0:Nov 14 14:06:03.991 UTC: card_mgr[2541]: %INFRA-FPD_Driver-6-UPGRADE_RESULT : Upgrade completes 93 percent for fpd Bootloader@location 0/RP0.
0/RP0/ADMIN0:Nov 14 14:06:08.922 UTC: card_mgr[2541]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader@0/RP0 image programming completed with UPGRADE DONE state Info: [Programming Done]
0/RP0/ADMIN0:Nov 14 14:06:08.923 UTC: card_mgr[2541]: %INFRA-FPD_Driver-1-UPGRADE_ALERT : FPD Bootloader @location 0/RP0 upgrade completed.
0/RP1/ADMIN0:Nov 14 14:06:08.924 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_SW_OPERATIONAL : Card: 0/RP0 software state going to Operational
0/RP1/ADMIN0:Nov 14 14:06:08.924 UTC: shelf_mgr[2783]: %INFRA-SHELF_MGR-6-CARD_HW_OPERATIONAL : Card: 0/RP0 hardware state going to Operational

RP/0/RP0/CPU0:5508#show install request

The install service operation 9 is 20% complete
RP/0/RP0/CPU0:5508#
RP/0/RP0/CPU0:Nov 14 14:07:54.516 UTC: sdr_instmgr[1188]: %MGBL-SCONBKUP-6-INTERNAL_INFO : Reload debug script successfully spawned
RP/0/RP1/CPU0:Nov 14 14:07:54.527 UTC: sdr_instmgr[1188]: %INSTALL-INSTMGR-2-OPERATION_SUCCESS : Install operation 97 finished successfully
Nov 14 14:07:56 Install operation 97 finished successfully
RP/0/RP1/CPU0:Nov 14 14:07:56.186 UTC: sdr_instmgr[1188]: %INSTALL-INSTMGR-2-SYSTEM_RELOAD_INFO : The whole system will be reloaded to complete install operation 97
0/RP1/ADMIN0:Nov 14 14:07:58.320 UTC: inst_mgr[3659]: %INFRA-INSTMGR-5-OPERATION_TO_RELOAD : This rack will now reload as part of the install operation
0/RP1/ADMIN0:Nov 14 14:08:47.794 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 0 state changed to DOWN
0/RP1/ADMIN0:Nov 14 14:08:47.794 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 1 state changed to DOWN
0/RP1/ADMIN0:Nov 14 14:08:47.794 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 2 state changed to DOWN
0/RP1/ADMIN0:Nov 14 14:08:47.794 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 3 state changed to DOWN
0/RP1/ADMIN0:Nov 14 14:08:47.794 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 4 state changed to DOWN
0/RP1/ADMIN0:Nov 14 14:08:47.795 UTC: fsdbagg[3629]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3629] : Plane 5 state changed to DOWN
0/FC2/ADMIN0:Nov 14 14:08:47.784 UTC: cm[1862]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/FC0/ADMIN0:Nov 14 14:08:47.781 UTC: cm[1866]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/FC4/ADMIN0:Nov 14 14:08:47.786 UTC: cm[1873]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/2/ADMIN0:Nov 14 14:08:47.800 UTC: cm[2508]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/FC1/ADMIN0:Nov 14 14:08:47.788 UTC: cm[1880]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/FC3/ADMIN0:Nov 14 14:08:47.784 UTC: cm[1859]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/RP0/ADMIN0:Nov 14 14:08:47.805 UTC: cm[2548]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/FC5/ADMIN0:Nov 14 14:08:47.786 UTC: cm[1869]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/SC0/ADMIN0:Nov 14 14:08:47.826 UTC: cm[1870]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.
0/SC1/ADMIN0:Nov 14 14:08:47.833 UTC: cm[1857]: %ROUTING-TOPO-5-PROCESS_UPDATE : Got process update: Card shutdown.</code>
</pre>
</div>  

The router reloads at this point.

<div class="highlighter-rouge">
<pre class="highlight">
<code>Stopping OpenBSD Secure Shell server: sshdinitctl: Unknown instance:
Stopping system message bus: dbus.
Stopping random number generator daemon.
Stopping system log daemon...0
Stopping kernel log daemon...0
Stopping internet superserver: xinetd.
Stopping crond: OK
Stopping rpcbind daemon...
done.
Stopping S.M.A.R.T. daemon: smartd.
Stopping Lighttpd Web Server: stopped /usr/sbin/lighttpd (pid 2342)
lighttpd.
Stopping libvirtd daemon: [  OK  ]
Deconfiguring network interfaces... done.
Sending all processes the KILL signal...
Unmounting remote filesystems...
Deactivating swap...
Unmounting local filesystems...
mount: can't find /mnt/ram in /etc/fstab
Rebooting... [223559.311086] obfl_rb_notify_cb: reboot notifier called
BIOS Ver: 09.30 Date: 07/11/2019 11:10:50

Press DEL or ESC to enter boot manager.                                     GRUB Secure Boot Validation Result: PASSED
[223561.069300] Requesting remote cold reset on 0/FC2 ...Requesting remote cold reset on 0/FC3 ...
[223561.182530] Requesting remote cold reset on 0/FC4 ...Requesting remote cold reset on 0/FC5 ...
GNU GRUB version 2.00sting remote cold reset on 0/SC0 ...Requesting remote cold reset on 0/SC1 ...
Press F2 to goto grub Menu..emote cold reset on 0/RP1 ...[223561.478902] Requesting local cold reset on 0/RP0 ...
Booting from Disk..cesfully wrote mtdoops at 0 size 32768
Loading Kernel..pstore: Successfully logged oops info. Size 32740
Kernel Secure Boot Validation Result: PASSED
Loading initrd..pervisor/RP Module
Initrd Secure Boot Validation Result: PASSED
[    0.234639] Allocating netns hash table
Enable selinux to relabel filesystem from initramfs
/usr/bin/chcon: cannot access '/dev/console': No such file or directory
Checking SELinux security contexts:
 * First booting, filesystem will be relabeled...
[*] Load IMA appraise policy: OK
Switching to new root and running init.
Sourcing /etc/sysconfig/udev20160406
Starting udev: [  OK  ]r: 0x80000000
/sbin/restorecon:  lstat(/etc/resolv.conf) failed:  No such file or directory
/sbin/restorecon:  lstat(/etc/adjtime) failed:  No such file or directory
Running postinst /etc/rpm-postinsts/100-dnsmasq...
Running postinst /etc/rpm-postinsts/101-nfs-utils-client...
Running postinst /etc/rpm-postinsts/102-nfs-utils...
update-rc.d: /etc/init.d/run-postinsts exists during rc.d purge (continuing)
 Removing any system startup links for run-postinsts ...
  /etc/rcS.d/S99run-postinsts
pd_notify_host_started
Sending Host OS Started event
Last Reset Reason =  0x80000000
Watchdog Register =  0x000000FE
SPI Boot Timer Register =  0x000060FF
Configuring network interfaces... done.
Starting system message bus: dbus.
UBI device number 3, total 144 LEBs (18855936 bytes, 18.0 MiB), available 0 LEBs (0 bytes), LEB size 130944 bytes (127.9 KiB)
Punching IOFPGA watchdog
Starting OpenBSD Secure Shell server: sshd
  generating ssh RSA key...
  generating ssh ECDSA key...
  generating ssh DSA key...
  generating ssh ED25519 key...
sshd start/running, process 3300
Starting rpcbind daemon...done.
Starting kdump:[  OK  ]
Starting random number generator daemonUnable to open file: /dev/tpm0
.
Starting system log daemon...0
Starting kernel log daemon...0
Starting HPA's tftpd: in.tftpd-hpa
.
Starting internet superserver: xinetd.
Starting S.M.A.R.T. daemon: smartd.
Starting Lighttpd Web Server: lighttpd.
Starting libvirtd daemon: [  OK  ]
Starting crond: OK
Starting cgroup-init
Network ieobc_br defined from /etc/init/ieobc_br_network.xml

Network local_br defined from /etc/init/local_br_network.xml

Network xr_local_br defined from /etc/init/xr_local_br_network.xml

Network ieobc_br started

Network local_br started

Network xr_local_br started

[*] ima_policy have loaded, or IMA policy file does not exist
mcelog start/running, process 5865
diskmon start/running, process 5866
Creating default host password file
initctl: Unknown

Connecting to 'default-sdr--1' console
��������bootlogd: ioctl(/dev/pts/2, TIOCCONS): Device or resource busy
/sbin/restorecon:  lstat(/etc/adjtime) failed:  No such file or directory
Running postinst /etc/rpm-postinsts/100-dnsmasq...
Running postinst /etc/rpm-postinsts/101-nfs-utils-client...
Running postinst /etc/rpm-postinsts/102-nfs-utils...
update-rc.d: /etc/init.d/run-postinsts exists during rc.d purge (continuing)
 Removing any system startup links for run-postinsts ...
  /etc/rcS.d/S99run-postinsts
Configuring network interfaces... done.
Starting system message bus: dbus.
Starting OpenBSD Secure Shell server: sshd
  generating ssh RSA key...
  generating ssh ECDSA key...
  generating ssh DSA key...
  generating ssh ED25519 key...
sshd start/running, process 2044
Starting rpcbind daemon...done.
Starting random number generator daemonUnable to open file: /dev/tpm0
.
Starting system log daemon...0
Starting kernel log daemon...0
Starting internet superserver: xinetd.
Libvirt not initialized for container instance
Starting crond: OK
SIOCADDRT: File exists
[*] ima_policy have loaded, or IMA policy file does not exist
Start serial incoming on , Clearing ..

DBG_MSG: platform type is 0
RP/0/RP0/CPU0:Nov 18 08:46:47.172 UTC: rmf_svr[324]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :RP-RED-LOST-NNR :DECLARE :0/RP0/CPU0:


ios con0/RP0/CPU0 is now available





Press RETURN to get started.





This product contains cryptographic features and is subject to United
States and local country laws governing import, export, transfer and
use. Delivery of Cisco cryptographic products does not imply third-party
authority to import, export, distribute or use encryption. Importers,
exporters, distributors and users are responsible for compliance with
U.S. and local country laws. By using this product you agree to comply
with applicable laws and regulations. If you are unable to comply with
U.S. and local laws, return this product immediately.

A summary of U.S. laws governing Cisco cryptographic products may be
found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to
export@cisco.com.



RP/0/RP0/CPU0:Nov 18 08:47:20.681 UTC: rmf_svr[324]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :RP-RED-LOST-NSRNR :DECLARE :0/RP0/CPU0:
0/RP1/ADMIN0:Nov 18 08:47:34.670 UTC: envmon[3173]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :Power Group redundancy lost :DECLARE :0:
0/RP0/ADMIN0:Nov 18 08:47:35.457 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-0 :DECLARE :: Fabric Plane-0 DOWN
0/RP0/ADMIN0:Nov 18 08:47:35.458 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-1 :DECLARE :: Fabric Plane-1 DOWN
0/RP0/ADMIN0:Nov 18 08:47:35.458 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-2 :DECLARE :: Fabric Plane-2 DOWN
0/RP0/ADMIN0:Nov 18 08:47:35.458 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-3 :DECLARE :: Fabric Plane-3 DOWN
0/RP0/ADMIN0:Nov 18 08:47:35.458 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-4 :DECLARE :: Fabric Plane-4 DOWN
0/RP0/ADMIN0:Nov 18 08:47:35.459 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-5 :DECLARE :: Fabric Plane-5 DOWN
RP/0/RP0/CPU0:Nov 18 08:47:36.013 UTC: ipv4_static[1030]: %ROUTING-IP_STATIC-4-CONFIG_NEXTHOP_ETHER_INTERFACE : Route for 19.0.0.0 is configured via ethernet interface without nexthop, Please check if this is intended
RP/0/RP0/CPU0:Nov 18 08:47:36.028 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_PM_ROLE_CHG : ISIS NSR PM role change, reg-type 0, HA-role: , 1
RP/0/RP0/CPU0:Nov 18 08:47:36.439 UTC: isis[1009]: %ROUTING-ISIS-6-INFO_STARTUP_START : Cold controlled start beginning
RP/0/RP0/CPU0:Nov 18 08:47:36.505 UTC: bpm[1057]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : bpm-active:(bgp-bpm-active)inst-id 0, Service Published
RP/0/RP0/CPU0:Nov 18 08:47:36.509 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONFIG_ENABLE : ISIS NSR is configured
RP/0/RP0/CPU0:Nov 18 08:47:36.515 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_NOTIFY_READY : ISIS NSR set nsr ISIS NSR not ready yet for instance ISIS_1. rc 0
0/RP0/ADMIN0:Nov 18 08:47:37.896 UTC: aaad[3158]: %MGBL-AAAD-7-DEBUG :  Not allowing to sync from XR VM to Admin VM after first user creation.
RP/0/RP0/CPU0:Nov 18 08:47:38.416 UTC: bpm[1057]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : bpm-default:(A)inst-id 0, Connection Open
RP/0/RP0/CPU0:Nov 18 08:47:38.668 UTC: bgp[1042]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : default, process instance 1:(A)inst-id 0, Connection Establised
RP/0/RP0/CPU0:Nov 18 08:47:39.135 UTC: cinetd[292]: %SECURITY-MPP-6-MSG_INFO : Updated Management Plane configuration for service: telnet
RP/0/RP0/CPU0:Nov 18 08:47:39.144 UTC: snmpd[1002]: %SECURITY-MPP-6-MSG_INFO : Updated Management Plane configuration for service: snmp


SYSTEM CONFIGURATION IN PROCESS


The startup configuration for this device is presently loading.
This may take a few minutes. You will be notified upon completion.

Please do not attempt to reconfigure the device until this process is complete.

Location : LAB ILM 3rd Floor

For operational problems contact :
Benoit des Rochettes                    email:  xxxxxxxx@cisco.com


RP/0/RP0/CPU0:Nov 18 08:47:39.414 UTC: bgp[1042]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : default:(A)inst-id 0, Initial Config Done

User Access Verification

Username: 0/RP0/ADMIN0:Nov 18 08:47:41.516 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-6-CARD_HW_OPERATIONAL : Card: 0/FC4 hardware state going to Operational
0/RP0/ADMIN0:Nov 18 08:47:41.516 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-6-CARD_SW_OPERATIONAL : Card: 0/FC4 software state going to Operational
0/RP0/ADMIN0:Nov 18 08:47:41.521 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-6-CARD_HW_OPERATIONAL : Card: 0/FC1 hardware state going to Operational
0/RP0/ADMIN0:Nov 18 08:47:41.521 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-6-CARD_SW_OPERATIONAL : Card: 0/FC1 software state going to Operational
0/RP0/ADMIN0:Nov 18 08:47:41.532 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-5-CARD_INSERTION : Location: 0/FC4, Serial #: SAL1945ST4F
0/RP0/ADMIN0:Nov 18 08:47:41.572 UTC: shelf_mgr[3203]: %INFRA-SHELF_MGR-5-CARD_INSERTION : Location: 0/FC1, Serial #: SAL1945ST3F
0/2/ADMIN0:Nov 18 08:47:43.989 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_ADMIN_UP : The admin state of the control ethernet switch port 7 has changed. New Admin state: UP, Link state UP
0/2/ADMIN0:Nov 18 08:47:43.989 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 7 has changed. New Link state UP, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:45.199 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_DOWN : The physical link state of the control ethernet switch port 7 has changed. New Link state DOWN, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:45.662 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_ADMIN_UP : The admin state of the control ethernet switch port 6 has changed. New Admin state: UP, Link state UP
0/2/ADMIN0:Nov 18 08:47:45.662 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 6 has changed. New Link state UP, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:46.862 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_DOWN : The physical link state of the control ethernet switch port 6 has changed. New Link state DOWN, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:47.615 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_ADMIN_UP : The admin state of the control ethernet switch port 5 has changed. New Admin state: UP, Link state UP
0/2/ADMIN0:Nov 18 08:47:47.615 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 5 has changed. New Link state UP, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:48.815 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_DOWN : The physical link state of the control ethernet switch port 5 has changed. New Link state DOWN, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:49.635 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_ADMIN_UP : The admin state of the control ethernet switch port 4 has changed. New Admin state: UP, Link state UP
0/2/ADMIN0:Nov 18 08:47:49.635 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 4 has changed. New Link state UP, Admin state: UP
0/2/ADMIN0:Nov 18 08:47:50.825 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_DOWN : The physical link state of the control ethernet switch port 4 has changed. New Link state DOWN, Admin state: UP

Username: cisco
Password:
RP/0/RP0/CPU0:Nov 18 08:47:55.841 UTC: exec[67856]: %SECURITY-LOGIN-6-AUTHEN_SUCCESS : Successfully authenticated user 'cisco' from 'console' on 'con0_RP0_CPU0'


0/FC1/ADMIN0:Nov 18 08:47:57.118 UTC: aaad[2021]: %MGBL-AAAD-7-DEBUG : Disaster-recovery account not configured. Using first user as disaster-recovery account
RP/0/RP0/CPU0:Nov 18 08:47:58.487 UTC: isis[1009]: %ROUTING-ISIS-6-INFO_STARTUP_FINISH : Cold controlled start completed
0/FC4/ADMIN0:Nov 18 08:47:58.654 UTC: aaad[2016]: %MGBL-AAAD-7-DEBUG : Disaster-recovery account not configured. Using first user as disaster-recovery account
0/2/ADMIN0:Nov 18 08:48:03.670 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 7 has changed. New Link state UP, Admin state: UP
LC/0/2/CPU0:Nov 18 08:48:07.116 UTC: fia_driver[177]: %PLATFORM-OFA-6-INFO : NPU #3 Initialization Completed
0/2/ADMIN0:Nov 18 08:48:07.264 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 6 has changed. New Link state UP, Admin state: UP
LC/0/2/CPU0:Nov 18 08:48:11.023 UTC: fia_driver[177]: %PLATFORM-OFA-6-INFO : NPU #2 Initialization Completed
0/2/ADMIN0:Nov 18 08:48:11.167 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 5 has changed. New Link state UP, Admin state: UP
LC/0/2/CPU0:Nov 18 08:48:14.605 UTC: fia_driver[177]: %PLATFORM-OFA-6-INFO : NPU #1 Initialization Completed
0/2/ADMIN0:Nov 18 08:48:14.754 UTC: esd[3180]: %INFRA-ESD-6-PORT_STATE_CHANGE_LINK_UP : The physical link state of the control ethernet switch port 4 has changed. New Link state UP, Admin state: UP
LC/0/2/CPU0:Nov 18 08:48:19.805 UTC: fia_driver[177]: %PLATFORM-OFA-6-INFO : NPU #0 Initialization Completed
RP/0/RP0/CPU0:Nov 18 08:48:25.472 UTC: pim[1223]: %ROUTING-IPV4_PIM-5-INTCHG : PIM interface Lo0 UP
RP/0/RP0/CPU0:Nov 18 08:48:25.472 UTC: pim[1223]: %ROUTING-IPV4_PIM-5-NBRCHG : PIM neighbor 75.75.127.1 UP on Lo0


SYSTEM CONFIGURATION COMPLETED

RP/0/RP0/CPU0:Nov 18 08:48:26.274 UTC: l2vpn_mgr[1241]: %L2-L2VPN-6-CAPABILITY_CHANGE : Global L2VPN capabilities have been updated
RP/0/RP0/CPU0:Nov 18 08:48:26.388 UTC: ifmgr[427]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Down
RP/0/RP0/CPU0:Nov 18 08:48:26.388 UTC: ifmgr[427]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface MgmtEth0/RP0/CPU0/0, changed state to Down
RP/0/RP0/CPU0:Nov 18 08:48:26.391 UTC: cfgmgr-rp[359]: %MGBL-CONFIG-6-OIR_RESTORE : Configuration for node '0/RP0/0' has been restored.
RP/0/RP0/CPU0:Nov 18 08:48:26.395 UTC: ifmgr[427]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Up
RP/0/RP0/CPU0:Nov 18 08:48:26.397 UTC: ifmgr[427]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface MgmtEth0/RP0/CPU0/0, changed state to Up
RP/0/RP0/CPU0:Nov 18 08:48:26.651 UTC: tacacsd[1193]: %SECURITY-TACACSD-6-SERVER_DOWN : TACACS+ server 192.0.0.2/49 is DOWN - Socket 116: No route to host
RP/0/RP0/CPU0:Nov 18 08:48:26.764 UTC: ifmgr[427]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Down
RP/0/RP0/CPU0:Nov 18 08:48:26.764 UTC: ifmgr[427]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface MgmtEth0/RP0/CPU0/0, changed state to Down
RP/0/RP0/CPU0:Nov 18 08:48:27.258 UTC: smartlicserver[144]: %LICENSE-SMART_LIC-5-COMM_RESTORED : Communications with Cisco licensing cloud restored
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/0, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/1, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/2, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/3, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/4, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/5, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/6, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/7, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/8, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/9, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/10, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/11, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/12, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/13, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/14, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/15, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/16, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/17, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/18, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/19, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/20, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/21, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/22, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/23, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/24, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/25, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/28, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/29, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/30, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/31, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/32, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/33, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/34, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.619 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/35, changed state to Down
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/0, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/1, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/2, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/3, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/4, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/5, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/6, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/7, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/14, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/15, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/16, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/17, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/20, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/28, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:29.630 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/29, changed state to Up
RP/0/RP0/CPU0:Nov 18 08:48:29.991 UTC: ifmgr[427]: %PKT_INFRA-LINK-3-UPDOWN : Interface MgmtEth0/RP0/CPU0/0, changed state to Up
RP/0/RP0/CPU0:Nov 18 08:48:29.992 UTC: ifmgr[427]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface MgmtEth0/RP0/CPU0/0, changed state to Up
LC/0/2/CPU0:Nov 18 08:48:33.543 UTC: fia_driver[177]: %PLATFORM-DPA-6-INFO : Fabric BANDWIDTH above configured threshold
0/RP0/ADMIN0:Nov 18 08:48:35.460 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-0 :CLEAR :: Fabric Plane-0 DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.461 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 0 state changed to UP
0/RP0/ADMIN0:Nov 18 08:48:35.462 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-1 :CLEAR :: Fabric Plane-1 DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.462 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 1 state changed to UP
0/RP0/ADMIN0:Nov 18 08:48:35.462 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 2 state changed to MCAST_DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.462 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-3 :CLEAR :: Fabric Plane-3 DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.463 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 3 state changed to UP
0/RP0/ADMIN0:Nov 18 08:48:35.463 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-4 :CLEAR :: Fabric Plane-4 DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.463 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 4 state changed to UP
0/RP0/ADMIN0:Nov 18 08:48:35.463 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-5 :CLEAR :: Fabric Plane-5 DOWN
0/RP0/ADMIN0:Nov 18 08:48:35.464 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 5 state changed to UP
0/RP0/ADMIN0:Nov 18 08:48:38.887 UTC: fsdbagg[3971]: %PKT_INFRA-FM-4-FAULT_MINOR : ALARM_MINOR :FABRIC-PLANE-2 :CLEAR :: Fabric Plane-2 MCAST_DOWN
0/RP0/ADMIN0:Nov 18 08:48:38.887 UTC: fsdbagg[3971]: %FABRIC-FSDB_AGG-5-PLANE_UPDOWN : [3971] : Plane 2 state changed to UP
RP/0/RP1/CPU0:Nov 18 08:48:39.108 UTC: ifmgr[314]: %PKT_INFRA-LINK-5-CHANGED : Interface MgmtEth0/RP1/CPU0/0, changed state to Administratively Down
RP/0/RP1/CPU0:Nov 18 08:48:39.114 UTC: cfgmgr-rp[139]: %MGBL-CONFIG-6-OIR_RESTORE : Configuration for node '0/RP1/0' has been restored.
RP/0/RP1/CPU0:Nov 18 08:48:39.316 UTC: cepki[426]: %SECURITY-PKI-6-LOG_INFO_DETAIL : FIPS POST Successful for  cepki
RP/0/RP1/CPU0:Nov 18 08:48:39.714 UTC: smartlicserver[392]: %LICENSE-SMART_LIC-6-AGENT_READY : Smart Agent for Licensing is initialized
RP/0/RP1/CPU0:Nov 18 08:48:39.888 UTC: smartlicserver[392]: %LICENSE-SMART_LIC-6-AGENT_ENABLED : Smart Agent for Licensing is enabled
RP/0/RP1/CPU0:Nov 18 08:48:39.888 UTC: smartlicserver[392]: %LICENSE-SMART_LIC-6-AGENT_READY : Smart Agent for Licensing is initialized
RP/0/RP1/CPU0:Nov 18 08:48:39.888 UTC: smartlicserver[392]: %LICENSE-SMART_LIC-6-AGENT_ENABLED : Smart Agent for Licensing is enabled
RP/0/RP1/CPU0:Nov 18 08:48:39.910 UTC: smartlicserver[392]: %LICENSE-SMART_LIC-6-HA_ROLE_CHANGED : Smart Agent HA role changed to Standby.
RP/0/RP1/CPU0:Nov 18 08:48:40.462 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_PM_ROLE_CHG : ISIS NSR PM role change, reg-type 0, HA-role: , 2
RP/0/RP1/CPU0:Nov 18 08:48:41.652 UTC: ipsec_mp[362]: %SECURITY-IMP-6-PROC_READY : Process ipsec_mp is ready
RP/0/RP1/CPU0:Nov 18 08:48:42.835 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_INTF_TOPO_UNREADY : ISIS NSR vm 2 intf init 'topo' not ready, retry later
RP/0/RP1/CPU0:Nov 18 08:48:42.835 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONFIG_ENABLE : ISIS NSR is configured
RP/0/RP1/CPU0:Nov 18 08:48:42.837 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONFIG_ENABLE : ISIS NSR is nsr ltopo ready triggered
RP/0/RP0/CPU0:Nov 18 08:48:42.840 UTC: bpm[1057]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : bpm-default:(S)inst-id 0, Connection Open
RP/0/RP1/CPU0:Nov 18 08:48:42.914 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONFIG_ENABLE : ISIS NSR is nsr ltopo ready triggered
RP/0/RP1/CPU0:Nov 18 08:48:43.024 UTC: isis[1009]: %ROUTING-ISIS-4-NSR_NII_BRINGUP : ISIS NSR NII intf 'Nii0' bring up timer running
RP/0/RP1/CPU0:Nov 18 08:48:43.024 UTC: isis[1009]: %ROUTING-ISIS-4-NSR_NII_BRINGUP : ISIS NSR NII intf 'Nii0' bring up timer running
RP/0/RP1/CPU0:Nov 18 08:48:43.028 UTC: snmpd[1002]: %SNMP-SNMP-3-INTERNAL : Snmp Trap source issue : Using best possible Src IP address for SNMP Traps. Failed to get the IP address of :(Check the Interface state):FortyGigE0_2_0_20
RP/0/RP1/CPU0:Nov 18 08:48:43.043 UTC: snmpd[1002]: %SNMP-SNMP-3-INTERNAL : Snmp Trap source issue : Using best possible Src IPV6 address for SNMP Traps. Failed to get the IPv6 address of :(Check the Interface state):FortyGigE0_2_0_20
RP/0/RP1/CPU0:Nov 18 08:48:43.447 UTC: bgp[1042]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : default, process instance 1:(S)inst-id 0, Connection Establised
RP/0/RP1/CPU0:Nov 18 08:48:43.800 UTC: bgp[1042]: %ROUTING-BGP-5-ASYNC_IPC_STATUS : default:(S)inst-id 0, Initial Config Done
RP/0/RP0/CPU0:Nov 18 08:48:45.353 UTC: rmf_svr[324]: %HA-REDCON-1-STANDBY_READY : standby card is ready
RP/0/RP1/CPU0:Nov 18 08:48:45.359 UTC: rmf_svr[359]: %HA-REDCON-6-STBY_STANDBY_READY : This card is standby and is ready
RP/0/RP0/CPU0:Nov 18 08:48:46.007 UTC: rmf_svr[324]: %HA-REDCON-1-STANDBY_NOT_READY : standby card is NOT ready
RP/0/RP1/CPU0:Nov 18 08:48:46.013 UTC: rmf_svr[359]: %HA-REDCON-1-STANDBY_NOT_READY : standby card is NOT ready
RP/0/RP0/CPU0:Nov 18 08:48:56.007 UTC: rmf_svr[324]: %HA-REDCON-1-STANDBY_READY : standby card is ready
RP/0/RP0/CPU0:Nov 18 08:49:00.158 UTC: rmf_svr[324]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :RP-RED-LOST-NNR :CLEAR :0/RP0/CPU0:
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/0, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/1, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/2, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/3, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/0, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/1, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/2, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.457 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/3, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/0, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/1, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/2, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/26/3, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/0, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/1, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/2, changed state to Up
LC/0/2/CPU0:Nov 18 08:49:07.459 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface Optics0/2/0/27/3, changed state to Up
RP/0/RP0/CPU0:Nov 18 08:49:09.891 UTC: isis[1009]: %ROUTING-ISIS-4-NSR_NII_BRINGUP : ISIS NSR NII intf 'Nii0' bring up timer running
RP/0/RP0/CPU0:Nov 18 08:49:09.898 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONN : ISIS NSR Connection 'Up' on 'Nii0'
RP/0/RP1/CPU0:Nov 18 08:49:09.905 UTC: isis[1009]: %ROUTING-ISIS-5-NSR_CONN : ISIS NSR Connection 'Up' on 'Nii0'
RP/0/RP0/CPU0:Nov 18 08:49:16.416 UTC: ztp.sh[65792]: %OS-SYSLOG-6-LOG_INFO : ZTP will abort as username is configured. Do not configure until ZTP aborted message is seen.
RP/0/RP0/CPU0:Nov 18 08:49:20.941 UTC: ztp.sh[65825]: %OS-SYSLOG-6-LOG_INFO : ZTP aborted

RP/0/RP0/CPU0:Nov 18 08:49:25.008 UTC: kim[278]: %INFRA-KIM-6-LOG_INFO : XR statistics will be pushed into the Linux kernel at 5 second intervals
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-3-UPDOWN : Interface HundredGigE0/2/0/0, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINEPROTO-5-UPDOWN : Line protocol on Interface HundredGigE0/2/0/0, changed state to Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/35, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/34, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/33, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/32, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/31, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/30, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/29, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/28, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/25, changed state to Administratively Down
LC/0/2/CPU0:Nov 18 08:49:27.528 UTC: ifmgr[223]: %PKT_INFRA-LINK-5-CHANGED : Interface HundredGigE0/2/0/24, changed state to Administratively Down</code>
</pre>
</div>  

At this step, the traffic of the different services is impacted.

![Screen Shot 2020-10-12 at 7.18.30 PM.png]({{site.baseurl}}/images/Screen Shot 2020-10-12 at 7.18.30 PM.png){: .align-center}

We verify the release and we commit it to "freeze" the installation (remember: if you don't commit the install, the next reload will start the previous release).

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show install active summary
    Active Packages: 7
        ncs5500-xr-6.6.3 version=6.6.3 [Boot image]
        ncs5500-mpls-2.1.0.0-r663
        ncs5500-isis-2.2.0.0-r663
        ncs5500-mpls-te-rsvp-4.1.0.0-r663
        ncs5500-mcast-3.1.0.0-r663
        ncs5500-mgbl-3.0.0.0-r663
        ncs5500-k9sec-3.1.0.0-r663

RP/0/RP0/CPU0:5508#install commit
Nov 18 08:56:14 Install operation 7 started by cisco:
  install commit
Nov 18 08:56:15 Install operation will continue in the background
RP/0/RP0/CPU0:5508#
RP/0/RP0/CPU0:Nov 18 08:56:36.025 UTC: sdr_instmgr[1245]: %PKT_INFRA-FM-6-FAULT_INFO : INSTALL-IN-PROGRESS :CLEAR :0/RP0/CPU0: INSTALL_IN_PROGRESS Alarm : being CLEARED for the system
0/RP0/ADMIN0:Nov 18 08:56:37.022 UTC: inst_mgr[3978]: %PKT_INFRA-FM-6-FAULT_INFO : INSTALL-IN-PROGRESS :CLEAR :0/RP0: Calvados INSTALL_IN_PROGRESS Alarm : being CLEARED for the system
Nov 18 08:56:42 Install operation 7 finished successfully
RP/0/RP0/CPU0:Nov 18 08:56:42.528 UTC: sdr_instmgr[1245]: %INSTALL-INSTMGR-2-OPERATION_SUCCESS : Install operation 7 finished successfully
RP/0/RP0/CPU0:Nov 18 08:58:18.895 UTC: bgp[1042]: %ROUTING-BGP-5-NSR_STATE_CHANGE : Changed state to NSR-Ready
RP/0/RP0/CPU0:Nov 18 08:58:35.888 UTC: pim6[1224]: %ROUTING-IPV4_PIM-5-HA_NOTICE : NSR ready
RP/0/RP0/CPU0:Nov 18 08:58:38.326 UTC: pim[1223]: %ROUTING-IPV4_PIM-5-HA_NOTICE : NSR not ready

RP/0/RP0/CPU0:5508#show install committed summary
    Committed Packages: 7
        ncs5500-xr-6.6.3 version=6.6.3 [Boot image]
        ncs5500-mpls-2.1.0.0-r663
        ncs5500-isis-2.2.0.0-r663
        ncs5500-mpls-te-rsvp-4.1.0.0-r663
        ncs5500-mcast-3.1.0.0-r663
        ncs5500-mgbl-3.0.0.0-r663
        ncs5500-k9sec-3.1.0.0-r663

RP/0/RP0/CPU0:5508#</code>
</pre>
</div>  

We verify the firmware:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508#show hw-module fpd
                                                               FPD Versions
                                                               =================
Location   Card type        HWver FPD device       ATR Status   Running Programd
------------------------------------------------------------------------------
0/2       NC55-36X100G-A-SE 0.303 MIFPGA               CURRENT    0.03    0.03
0/2       NC55-36X100G-A-SE 0.303 Bootloader           CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 DBFPGA               CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 IOFPGA               CURRENT    0.21    0.21
0/2       NC55-36X100G-A-SE 0.303 SATA                 CURRENT    5.00    5.00
0/RP0     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
0/RP0     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/RP0     NC55-RP           1.1   SATA                 CURRENT    6.00    6.00
0/RP1     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
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
0/2       NC55-36X100G-A-SE 0.303 Bootloader           CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 DBFPGA               CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 IOFPGA               CURRENT    0.21    0.21
0/2       NC55-36X100G-A-SE 0.303 SATA                 CURRENT    5.00    5.00
0/RP0     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
0/RP0     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/RP0     NC55-RP           1.1   SATA                 CURRENT    6.00    6.00
0/RP1     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
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
0/SC1     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10</code>
</pre>
</div>

To make sure you are ready for the next step, one way could be check the service are restored. Another one, is to verify the "NSR ready" state in the show redundancy summary output. It shows that both RPs are in sync and the router is in stable/nominal state.

### Hardware migration

We are now ready for the fan trays and fabric cards replacement.  
At this point, it's recommended to perform the migration with the chassis shutdown electrically.

Unplug all power blocks:

![1-power.png]({{site.baseurl}}/images/1-power.png){: .align-center}

Unscrew the fan trays:

![2-FT1.png]({{site.baseurl}}/images/2-FT1.png){: .align-center}

Remove the fan trays one by one:

![3-FT2.png]({{site.baseurl}}/images/3-FT2.png){: .align-center}

Unscrew the fabric cards:

![4-FC1.png]({{site.baseurl}}/images/4-FC1.png){: .align-center}

Eject all fabric cards:

![5-FC.png]({{site.baseurl}}/images/5-FC.png){: .align-center}

Replace them with v2 fabric cards:

![5b-FC.png]({{site.baseurl}}/images/5b-FC.png){: .align-center}

Make sure your remove the plastic cache before insertion :)  
(you have no idea what we've seen...)

![FC2-cache.png]({{site.baseurl}}/images/FC2-cache.png){: .align-center}

Insert the three v2 fan trays:

**IMPORTANT**: Make sure you don't insert any v1 Fan with v2 Fabric or vice-versa. These parts are shouldn't allow the insertion (with a pin guide to avoid mistake), but don't force the insertion. If it doesn't fit, make sure you didn't mix up cards and fans.
{: .notice--info}

![6-FT3.png]({{site.baseurl}}/images/6-FT3.png){: .align-center}

At this point, we can re-plug the power blocks and watch the system booting up.

Once completed, we verify the parts inserted

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
0/FC0             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FC1             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FC2             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FC3             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FC4             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FC5             NC55-5508-FC2              OPERATIONAL       NSHUT
0/FT0             NC55-5508-FAN2             OPERATIONAL       NSHUT
0/FT1             NC55-5508-FAN2             OPERATIONAL       NSHUT
0/FT2             NC55-5508-FAN2             OPERATIONAL       NSHUT
0/SC0             NC55-SC                    OPERATIONAL       NSHUT
0/SC1             NC55-SC                    OPERATIONAL       NSHUT
RP/0/RP0/CPU0:5508#</code>
</pre>
</div>

And the firmware:

<div class="highlighter-rouge">
<pre class="highlight">
<code>sysadmin-vm:0_RP0# show hw-module fpd
                                                                   FPD Versions
                                                                ===============
Location  Card type         HWver FPD device       ATR Status   Run    Programd
-------------------------------------------------------------------------------
0/2       NC55-36X100G-A-SE 0.303 Bootloader           CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 DBFPGA               CURRENT    0.14    0.14
0/2       NC55-36X100G-A-SE 0.303 IOFPGA               CURRENT    0.21    0.21
0/2       NC55-36X100G-A-SE 0.303 SATA                 CURRENT    5.00    5.00
0/RP0     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
0/RP0     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/RP0     NC55-RP           1.1   SATA                 CURRENT    6.00    6.00
0/RP1     NC55-RP           1.1   Bootloader           CURRENT    9.30    9.30
0/RP1     NC55-RP           1.1   IOFPGA               CURRENT    0.09    0.09
0/FC0     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC0     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/FC1     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC1     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/FC2     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC2     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/FC3     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC3     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/FC4     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC4     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/FC5     NC55-5508-FC2     1.0   Bootloader           CURRENT    1.80    1.80
0/FC5     NC55-5508-FC2     1.0   IOFPGA               CURRENT    0.12    0.12
0/SC0     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC0     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10
0/SC1     NC55-SC           1.4   Bootloader           CURRENT    1.74    1.74
0/SC1     NC55-SC           1.4   IOFPGA               CURRENT    0.10    0.10</code>
</pre>
</div>

Very likely you'll have to update the firmware for the new v2 parts and reload them to make it active.

## Other show commands you could use

In no specific order, you could have a look at the output of the following additional show command before and after the software upgrade and hardware migration.

- (admin) show controller card-mgr inventory
- (admin) show led
- (admin) show environment all
- (admin) show alarms
- show filesystem location 0/rp0/CPU0
- show filesystem location 0/rp1/CPU0	
- show process cpu
- show processes blocked (several times to avoid transcient blocks)
- (admin) show controller fabric plane all
- (admin) show controller fabric health
- (admin) show controller fabric plane all statistics 
- (admin) show controller fabric plane all detail

## Configuration used for the test

Mostly for reference

### TREX configs

TREX-Config-port0.yaml

<div class="highlighter-rouge">
<pre class="highlight">
<code>- name: L3VPN-1
  stream:
    name: L3VPN-1
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 0
    action_count: 0
    packet:
      meta: ''
      binary: >-
        AAAAAAAAAAAAAAAAgQAD6AgARQAFxgABAABABl4uCwAAAgwAAAInEE4gAAAAAAAAAABQAiAA/hAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 10000
          init_value: 10000
          step: 1
          max_value: 11000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1500
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 100
      rule_type: stats
- name: L2VPN-1
  stream:
    name: L2VPN-1
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 2
    action_count: 0
    packet:
      meta: ''
      binary: >-
        PP3+pKriAAAAAAAAgQAAZAgARQAFxgABAABABjUwEAAAATAAAAFOIHUwAAAAAAAAAABQAiAAhvIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 20000
          init_value: 20000
          step: 1
          max_value: 21000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1400
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 200
      rule_type: stats
- name: L2VPN-RSVP-1
  stream:
    name: L2VPN-RSVP-1
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 2
    action_count: 0
    packet:
      meta: ''
      binary: >-
        PP3+pKriAAAAAAAAgQAAZQgARQAFYgABAABABjWUEAAAATAAAAFOIHUwAAAAAAAAAABQAiAAh1YAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 20000
          init_value: 20000
          step: 1
          max_value: 21000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1400
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 300
      rule_type: stats
</code>
</pre>
</div>

and TREX-Config-port1.yaml

<div class="highlighter-rouge">
<pre class="highlight">
<code>- name: L3VPN-2
  stream:
    name: L3VPN-2
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 0
    action_count: 0
    packet:
      meta: ''
      binary: >-
        AAAAAAAAAAAAAAAAgQAH0AgARQAFxgABAABABl4uDAAAAgsAAAInECcQAAAAAAAAAABQAiAAJSEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 10000
          init_value: 10000
          step: 1
          max_value: 11000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1500
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 101
      rule_type: stats
- name: L2VPN-2
  stream:
    name: L2VPN-2
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 2
    action_count: 0
    packet:
      meta: ''
      binary: >-
        PP3+pKrhAAAAAAAAgQAAZQgARQAFxgABAABABjUwMAAAARAAAAFOIJxAAAAAAAAAAABQAiAAX+IAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 20000
          init_value: 20000
          step: 1
          max_value: 21000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1400
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 201
      rule_type: stats
- name: L2VPN-RSVP-2
  stream:
    name: L2VPN-RSVP-2
    advanced_mode: true
    enabled: true
    self_start: true
    next_stream_id: -1
    isg: 0
    flags: 2
    action_count: 0
    packet:
      meta: ''
      binary: >-
        PP3+pKrhAAAAAAAAgQAAZggARQAFYgABAABABjWUMAAAARAAAAFOIJxAAAAAAAAAAABQAiAAYEYAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA==
      model: ''
    rx_stats:
      enabled: false
    vm:
      split_by_var: ''
      instructions:
        - next_var: null
          name: TCP_sport_5
          min_value: 20000
          init_value: 20000
          step: 1
          max_value: 21000
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - next_var: null
          name: pkt_size_1
          min_value: 80
          init_value: 80
          step: 1
          max_value: 1400
          type: flow_var
          size: 2
          op: inc
          split_to_cores: true
        - name: TCP_sport_5
          add_value: 0
          pkt_offset: 38
          type: write_flow_var
          is_big_endian: true
        - type: trim_pkt_size
          name: pkt_size_1
        - name: pkt_size_1
          add_value: -18
          pkt_offset: 20
          type: write_flow_var
          is_big_endian: true
        - type: fix_checksum_ipv4
          pkt_offset: 18
      cache_size: 5000
    mode:
      rate:
        type: pps
        value: 1
      type: continuous
      count: 1
      pkts_per_burst: 1
      total_pkts: 1
      ibg: 0
    flow_stats:
      enabled: true
      stream_id: 301
      rule_type: stats
</code>
</pre>
</div>

### Router under test config

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

## Acknowledgement

Tons of thanks to Benoit who made himself available during vacations for the video recording and for providing such a detailed MOP.
