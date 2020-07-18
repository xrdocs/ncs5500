---
published: false
date: '2020-07-18 15:14 +0200'
title: BGP FlowSpec to ACL Script
---
{% include toc icon="table" title="BGPFS2ACL" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Introduction

BGP Flowspec is a technology we described in multiple posts and videos:  
- [https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/](https://xrdocs.io/ncs5500/tutorials/bgp-flowspec-on-ncs5500/)
- [Cisco NCS5500 Flowspec (Principles and Configuration) Part1](https://www.youtube.com/watch?v=dTgh0p9Vyns)
- [Cisco NCS5500 Flowspec (Auto-Mitigation of a Memcached Attack) Part2](https://www.youtube.com/watch?v=iRPob7Ws2v8)

In July 2020, we support BGP Flowspec only on the NCS5500 products equiped with the following ASICs and memories:
- Jericho+ with OP eTCAM
	- NCS55A1-36H-SE-S
    - NCS55A2-MOD-SE-S
    - NC55-36X100G-A-SE
- Jericho2 with OP2 eTCAM
	- NC57-18DD-SE

All these systems and line cards can program the flowspec rules in the datapath, matching packets based on a description and applying actions like drop, rate-limit, remark dscp, redirect to IP or to VRF. But the rest of the portfolio can't support it.   
Nevertheless, it's possible to configure a BGP FS client on all the IOS XR, even if they are not able to program the hardware leveL.  
That's an useful feature because we can develop a program executed on the router itself to convert the flowspec rules received into configuration line. That's exactly what the bgpfs2acl script is doing.  
Routers powered by the Jericho+ with large LPM (NCS55A1-24H, NCS55A1-48Q6H, NCS55A1-24Q6H-SS) are used in peering position and will be perfect candidates for this script.

Thanks to Carl Fredrik Lagerfeldt and Johan Gustawsson who brought first the original idea.

## Video

<iframe type="text/html" width="560" height="315" src="https://www.youtube.com/embed/8pNZJUHlRPk?autoplay=1" frameborder="0" allow="autoplay" ></iframe>{: .align-center}

In this interview, Mike explained the basic tools used by the script to operate (ZTP config, etc). Since the day of this interview, Dmitrii Rudnev worked on the script and changed the structure entirely, extending the capabilities significantly.

## The script

### Github

Code is available on Github: [https://github.com/ios-xr/bgpfs2acl](https://github.com/ios-xr/bgpfs2acl).

### Description / Match

In this release, it covers most description options with the following exceptions:  
- no match on DSCP field
- no match on TCP flag
- match on ICMP type and code works but no support of lists or ranges in type and code
- no match on Dont Fragment

### Action

In term of action, it supports drop and redirect-to-IP but not:
- set dscp
- rate limit to X bps
- redirect to VRF

### Support of packet-length ranges

To support the range in packet length, it's necessary to enable a specific UDK configuration:

<div class="highlighter-rouge">
<pre class="highlight">
<code>hw-module profile tcam format access-list ipv4 src-addr dst-addr src-port dst-port proto packet-length frag-bit port-range</code>
</pre>
</div>

Starting up the script, it will check the presence of this config line and warn it will not be able to "translate" rules containing length ranges if it's not enabled.  
If it receives such a rule and the config is not present, it will trigger a message to inform the rule has not been handled.  

Note: enabling this command requires a reload of the line card or chassis.
{: .notice—info}

### Support of fragments

ACL match on fragment-type is only supported on systems with external TCAM because it requires to enable the compression feature. It goes against the main purpose of the script, which is to mimic the flowspec behavior on non-eTCAM routers.  
But matching fragments (without going in deeper level of subtleties like First Fragment, Last Fragment, Is Fragmented, etc) is supported, that why all frag-types described in the flowspec rule will be translated into the same ACL line "fragments".  

## Tests

### Example1: Destination IP and UDP source port

On the controler:

<div class="highlighter-rouge">
<pre class="highlight">
<code>class-map type traffic match-all CHARGEN
   match destination-address ipv4 7.7.7.7 255.255.255.255
   match protocol udp
   match source-port 19
policy-map type pbr Example1
 class type traffic CHARGEN
  drop
 !
!
flowspec
 address-family ipv4
  service-policy type pbr Example1
!</code>
</pre>
</div>

Before the receiving the BGP FS rule on the client side:

<div class="highlighter-rouge">
<pre class="highlight">
<code>interface TenGigE0/0/0/1
 ipv4 address 44.55.66.77 255.255.255.0
 ipv4 access-group test2 ingress
!
interface TenGigE0/0/0/2
 ipv4 address 55.66.77.88 255.255.255.0
 ipv4 access-group bgpfs2acl-ipv4 ingress
!</code>
</pre>
</div>

These two ACLs are applied to two different interfaces.

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>

<div class="highlighter-rouge">
<pre class="highlight">
<code></code>
</pre>
</div>


