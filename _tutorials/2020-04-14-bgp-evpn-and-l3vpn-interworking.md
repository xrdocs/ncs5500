---
published: true
date: '2020-04-14 16:35 -0700'
title: BGP-EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP-EVPN and L3VPN Interworking Support on NCS5500

BGP EVPN and L3VPN interworking is a way to connect EVPN domain such as a DC or CO over an IPVPN network. This is generally needed when we want end-to-end connectivity of Hosts/CEs in EVPN domain to connect to other domains over an IPVPN network providing inter-subnet routing.
Below topology shows an EVPN domain connecting to L3VPN domain, where interworking is done on Boarder-Leaf/DCI routers. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to domain and send summarized advertisement out.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-topology.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will see the Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s loopback-100 prefix learnt on Leafs. This will provide end-to-end reachability between Hosts and PE-1’s prefixes, that are advertised by different BGP address families. The configuration setup is based on single BGP AS 65001 but separate ISIS routing domains for EVPN and L3VPN. DCI is participating in both ISIS domains performing EVPN and L3VPN interworking.

As we look at the setup, there are four major tasks we have to implement to achieve end-to-end connectivity. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below visit previous posts. 
•	Configure Segment Routing
•	Configure EVPN Layer-2 Service and IRB
•	Configure L3VPN
•	Configure DCI and Leafs to support EVPN and L3VPN interworking


### Task:1 Configuration of Segment Routing on DCI.
Segment routing configuration is covered in earlier post. For this we will show segment routing configuration of DCI as DCI is part of two segment routing domains. One segment routing domain is providing forwarding to EVPN fabric and the other to L3VPN domain.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/evpn-l3vpn-interworking-transport.png?raw=true)

<table border="1">
  <tr>	<th><b>DCI-1</b>	</th>	<th><b>	DCI-2	</b></th>	</tr>
<tr border="0">	<th>	DCI-1#show isis segment-routing label table 	</th>	<th>	DCI-2#show isis segment-routing label table 	</th>	</tr>
<tr border="0">	<th>		</th>	<th>		</th>	</tr>
<tr>	<th>	IS-IS 1 IS Label Table	</th>	<th>	IS-IS 1 IS Label Table	</th>	</tr>
<tr>	<th>	Label         Prefix/Interface	</th>	<th>	Label         Prefix/Interface	</th>	</tr>
<tr>	<th>	----------    ----------------	</th>	<th>	----------    ----------------	</th>	</tr>
<tr>	<th>	16001         1.1.1.1/32	</th>	<th>	16001         1.1.1.1/32	</th>	</tr>
<tr>	<th>	16002         2.2.2.2/32	</th>	<th>	16002         2.2.2.2/32	</th>	</tr>
<tr>	<th>	16006         6.6.6.6/32	</th>	<th>	16006         6.6.6.6/32	</th>	</tr>
<tr>	<th>	16007         7.7.7.7/32	</th>	<th>	16007         7.7.7.7/32	</th>	</tr>
<tr>	<th>	16008         Loopback0	</th>	<th>	16008         8.8.8.8/32	</th>	</tr>
<tr>	<th>	16009         9.9.9.9/32	</th>	<th>	16009         Loopback0	</th>	</tr>
<tr>	<th>		</th>	<th>		</th>	</tr>
<tr>	<th>	IS-IS 2 IS Label Table	</th>	<th>	IS-IS 2 IS Label Table	</th>	</tr>
<tr>	<th>	Label         Prefix/Interface	</th>	<th>	Label         Prefix/Interface	</th>	</tr>
<tr>	<th>	----------    ----------------	</th>	<th>	----------    ----------------	</th>	</tr>
<tr>	<th>	16008         Loopback0	</th>	<th>	16008         8.8.8.8/32	</th>	</tr>
<tr>	<th>	16009         9.9.9.9/32	</th>	<th>	16009         Loopback0	</th>	</tr>
<tr>	<th>	16010         10.10.10.10/32 	</th>	<th>	16010         10.10.10.10/32	</th>	</tr>
<tr>	<th>		</th>	<th>		</th>	</tr>
</table>





<table border="1">
  <tr>
    <th>Month</th>
    <th>Savings</th>
  </tr>
  <tr>
    <td>January</td>
    <td>$100</td>
  </tr>
  <tr>
    <td>February</td>
    <td>$80</td>
  </tr>
</table>


| **Loopback 0** | **Prefix-SID** | **ISIS Net** |
| Spine-1 6.6.6.6/32 | 16006 | 49.0001.0000.0000.0006.0 |
| Spine-2 7.7.7.7/32 | 16007 | 49.0001.0000.0000.0007.0 |
| Leaf-1  1.1.1.1/32 | 16001 | 49.0001.0000.0000.0001.0 |
| Leaf-2  2.2.2.2/32 | 16002 | 49.0001.0000.0000.0002.0 |
| Leaf-5  5.5.5.5/32 | 16005 | 49.0001.0000.0000.0005.0 |

<table border="1">
  <tr>
    <th>DCI-1</th>
    <th>DCI-2</th>
  </tr>
  <tr>
    <th>
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
      show ip int br
      </code>
      </pre>
      </div>
    </th>
    <th>
      <div class="highlighter-rouge">
      <pre class="highlight">
      <code>
      show ip int br
      </code>
      </pre>
      </div>
    </th>
  </tr>
</table>
  