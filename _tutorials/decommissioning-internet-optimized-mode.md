---
published: true
date: '2020-12-10 13:29 +0100'
title: Decommissioning the Internet-Optimized Mode
author: Nicolas Fevrier
excerpt: Brief note on the removal of the internet-optimized mode
position: top
---
{% include toc icon="table" title="Remove Internet-opt Mode" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

**Update**: Contrary to what was initially described, the decommissionning in 7.3.1 only covers the IPv4 CLI. The v6 CLI will be removed in 7.4.1.
{: .notice--info}

## Introduction

In early 2021, with the IOS XR release 7.3.1, we will remove the internet-optimized mode.  
This feature was introduced to optimize the v4/v6 prefix distribution between LEM and LPM for the first generation of NCS5500 products. 

## Feature description

Multiple hacks have been used for this optimization, changing the lookup order of operation, spliting the /23 prefixes in two sub-sequent /24s, ...  
During years, it was working fine for internet distribution as demonstrated in December 2017 in this blog post: [https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/](https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/).  

**So why do you want to remove this feature?**

Multiple reasons:  
- full internet view doesn't fit anymore: we started passing the message two years ago in CiscoLive. With large IGP, MPLS and MAC addresses: it doesn't work any more.  
Internet v4 and v6 is growing too vast and the optimization if not enough. We don't recommend to use systems without eTCAM, or at least without large LPM (like the NCS55A1-24H and the NCS55A1-24Q-6H-SS, but not the -S) if you need a full internet view.  
- it was not widely deployed. We identified just a small amount of customers enabling that feature.
- it's using space in the code that could be re-allocated to something new

## Decommissioning

Starting from IOS XR 7.3.1 (a release targeted for Feb/Mar 2021), we will remove this hw-module configuration options:
- hw-module fib ipv4 scale internet-optimized

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-1-721(config)#hw-module fib ipv4 scale ?
  host-optimized-disable  Configure Host optimization by default
  <mark><del>internet-optimized      Configure Intetrnet optimized</del></mark>
RP/0/RP0/CPU0:5508-1-721(config)</code>
</pre>
</div>

If you perform an upgrade to 7.3.1 and the config lines are present, it will be displayed as:  

<div class="highlighter-rouge">
<pre class="highlight">
<code> "Pre-existing [hw-module fib ipv4 scale internet-optimized] config has "  
 "been found. This feature isn't supported anymore and therefore ignored. "  
 "Please delete this config. "  </code>
</pre>
</div>

But it's just a warning message, not an error.

What will be the remaining options:

### default: host-optimized mode

No configuration, the IPv4 and IPv6 will be sorted in the internal database following this logic:

![host-optimized-default.png]({{site.baseurl}}/images/host-optimized-default.png){: .align-center}


### configure-able: host-optimized-disable and custom-lem

The remaining configuration options will be:  
- v4 host-optimized-disable
- v6 custom-lem and internet-optimized

![host-disable-custom-lem.png]({{site.baseurl}}/images/host-disable-custom-lem.png){: .align-center}


## Document updated

We also added a mention in the following docs and videos to inform that internet-optimized mode will be removed and shouldn't be used anymore:

- [https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/](https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/)
- [https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/](https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/)
- [https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/](https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/)
- [https://xrdocs.io/ncs5500/tutorials/Understanding-ncs5500-jericho-plus-systems/](https://xrdocs.io/ncs5500/tutorials/Understanding-ncs5500-jericho-plus-systems/)
- [https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/](https://xrdocs.io/ncs5500/tutorials/ncs5500-urpf/)
- [https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/](https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/)
- [https://www.youtube.com/watch?v=8Tq4nyP2wuA](https://www.youtube.com/watch?v=8Tq4nyP2wuA)
- [https://www.youtube.com/watch?v=nT31rHqFm-o](https://www.youtube.com/watch?v=nT31rHqFm-o)
