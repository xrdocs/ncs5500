---
published: true
date: '2021-08-04 22:56 +0200'
title: Full Internet in Jericho2 NPU and Programming Speed
author: Nicolas Fevrier
position: hidden
excerpt: Understand the J2 memory utilization and the programming / convergence speed
---
{% include toc icon="table" title="Full Internet in J2" %}  

<iframe class="responsive" width="560" height="315" src="https://www.youtube.com/embed/-wGhup-ZTZ4" frameborder="0" allowfullscreen></iframe>{: .align-center}  
.

Articles mentioned in the video:

- Decommissionning the internet-optimized mode:  
[https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/](https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/)  
- NCS5500 Routing Resource with 2020 Internet (and Future)  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/](https://xrdocs.io/ncs5500/tutorials/ncs5500-routing-resource-with-2020-internet/)  
- NCS5500 FIB Programming Speed  
[https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/](https://xrdocs.io/ncs5500/tutorials/ncs5500-fib-programming-speed/)  

## NCS5700 and the Jericho2 NPU

The NCS5500 products series is composed of different routing devices, fixed port "pizza boxes" and modular chassis with line cards. To clearly identify the products based on Broadcom Jericho2 ASICs, we named the fixed platforms "NCS-57xx" and the line cards in the chassis "NC57-xx". It's also the case for J2C platforms.  
The suffix "-SE" is used to describe systems and line cards with external TCAM:

| PID | Port Density | NPU | eTCAM |
|:-----:|:-----:|:-----:|:-----:|
| NCS57B1-6D24 | 24x 100GE, 6x 400GE | 1x J2 | No |
| NCS57B1-5DSE | 24x 100GE, 5x 400GE | 1x J2 | Yes |
| NC57-24DD | 24x 400GE | 2x J2 | No |
| NC57-18DD-SE | 24x Flex Ports + 6x 400G | 2x J2 | Yes |
| NC57-36H6D-S | 36x Flex Ports | 1x J2 | No |
| NC57-36H-SE | 36x 100GE | 1x J2 | Yes |

Note: More platforms coming soon in IOS XR 7.4.1 and in following releases
{: .notice--danger}

### Internal Resources

Like their predecessors, the J2 NPUs are leveraging different internal memories to store information, including routing details.

If not equipped with eTCAM, we have LEM and LPM:

![J2.png]({{site.baseurl}}/images/J2.png){: .align-center}

If the NPU is completed by an external TCAM:

![J2-SE.png]({{site.baseurl}}/images/J2-SE.png){: .align-center}





------

![36x100G-SE.jpg]({{site.baseurl}}/images/36x100G-SE.jpg){: .align-center}

<div class="highlighter-rouge">
<pre class="highlight">
<code>

</code>
</pre>
</div>

Note: blabla
{: .notice--danger}

