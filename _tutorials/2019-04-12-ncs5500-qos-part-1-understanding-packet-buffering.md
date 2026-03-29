---
published: false
date: '2019-04-12 16:58 +0200'
title: NCS5500 QoS Part 1 - Understanding Packet Buffering
author: Nicolas Fevrier
excerpt: 'First part of the NCS5500 QoS Series: Packet Buffering'
---
{% include toc icon="table" title="NCS5500 Buffering Architecture" %} 

You can find more content related to NCS5500 including routing memory management, VRF, URPF, ACLs, Netflow following this [link](https://xrdocs.io/ncs5500/tutorials/).

## Cisco NCS5500 QOS Series - 01 - Introduction to Packet BUFFERING

This first blog post will help understanding key concepts on the buffering architecture and clarify things like VOQ-only, single-lookup and ingress-buffering only designs.  
It's necessary to detail all these mechanisms to understand later the subtleties of the QoS implementation, including the scales and limitations.

### Video

The short version of this article is available in the following Youtube video:

[https://www.youtube.com/watch?v=_ozPYN6Ej9Y](https://www.youtube.com/watch?v=_ozPYN6Ej9Y)

<iframe src="https://www.youtube.com/watch?v=_ozPYN6Ej9Y" width="800" height="550" frameborder="0" allowfullscreen webkitallowfullscreen msallowfullscreen></iframe>

### DNX ASICs

All the NCS5500 routers are powered by the Broadcom StrataDNX (or DNX) Network Processor Units (NPUs).  
These ASICs can be used in standalone mode (Systems on Chip) or interconnected through one or multiple Fabric Engines.  




![Base vs Scale]({{site.baseurl}}/images/base-scale.jpg){: .align-center}


<div class="highlighter-rouge">
<pre class="highlight">
<code>
CODE CODE CODE
<mark>NC55-36X100G</mark> 
CODE CODE CODE
</code>
</pre>
</div>

Notice-blablabla
{: .notice--info}


