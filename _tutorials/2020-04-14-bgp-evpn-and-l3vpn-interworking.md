---
published: false
date: '2020-04-14 16:35 -0700'
title: BGP-EVPN and L3VPN Interworking
author: Ahmad Bilal
excerpt: BGP-EVPN and L3VPN Interworking Support on NCS5500
---
{% include toc %}

## BGP-EVPN and L3VPN Interworking Support on NCS5500

BGP EVPN and L3VPN interworking is a way to connect EVPN domain such as a DC or CO over an IPVPN network. This is generally needed when we want end-to-end connectivity of Hosts/CEs in EVPN domain to connect to other domains over an IPVPN network providing inter-subnet routing.
Below topology shows an EVPN domain connecting to L3VPN domain, where interworking is done on Boarder-Leaf/DCI routers. Boarder-Leaf/DCI routers are essential in these types of designs to keep prefixes local to domain and send summarized advertisement out.

![](https://github.com/xrdocs/ncs5500/blob/gh-pages/images/evpn-config/reference-topology-evpn-single-active.png?raw=true)

In this post we will go over the configuration of EVPN and L3VPN interworking on NCS 5500 routers acting as DCI. When we complete the configuration, we will see the Host subnet (10.0.0.0/24) learnt on PE-1, and PE-1’s loopback-100 prefix learnt on Leafs. This will provide end-to-end reachability between Hosts and PE-1’s prefixes, that are advertised by different BGP address families. The configuration setup is based on single BGP AS 65001 but separate ISIS routing domains for EVPN and L3VPN. DCI is participating in both ISIS domains performing EVPN and L3VPN interworking.

As we look at the setup, there are four major tasks we have to implement to achieve end-to-end connectivity. Some of these tasks are already covered in previous posts, their details will not be covered here. Click on the links below visit previous posts. 
•	Configure Segment Routing
•	Configure EVPN Layer-2 Service and IRB
•	Configure L3VPN
•	Configure DCI and Leafs to support EVPN and L3VPN interworking


### Task:1 Configuration of Segment Routing on DCI.
Segment routing configuration is covered in earlier post. For this we will show segment routing configuration of DCI as DCI is part of two segment routing domains. One segment routing domain is providing forwarding to EVPN fabric and the other to L3VPN domain.

