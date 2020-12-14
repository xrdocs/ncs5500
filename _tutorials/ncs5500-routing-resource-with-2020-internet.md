---
published: true
date: '2020-07-26 15:04 +0200'
title: NCS5500 Routing Resource with 2020 Internet (and Future)
author: Nicolas Fevrier
excerpt: What a full internet table in 2020 consumes in the NCS5500?
tags:
  - iosxr
  - internet
  - bgp
  - ncs5500
position: top
---
{% include toc icon="table" title="NCS5500 and the 2020 Internet" %} 

![NCS 5500 INTERNAL RESOURCES.png]({{site.baseurl}}/images/NCS 5500 INTERNAL RESOURCES.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

Important Update: in IOS XR 7.3.1, we will decommission the "internet-optimized" mode, please check this article: [https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/](https://xrdocs.io/ncs5500/tutorials/decommissioning-internet-optimized-mode/)
{: .notice--danger}

## Introduction

Between September 2017 and March 2018, we published five articles to answer recurrent questions on the routing memory utilization. The NCS5500 adoption was ramping up, and users were learning about the LEM, LPM and eTCAM used to store the routing information.  
1 - NCS5500 Resources focusing on IPv4 prefixes:  
[https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/](https://xrdocs.io/ncs5500/tutorials/2017-08-03-understanding-ncs5500-resources-s01e02/)  
2 - NCS5500 Resources focusing on IPv6 prefixes:  
[https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/](https://xrdocs.io/ncs5500/tutorials/2017-08-07-understanding-ncs5500-resources-s01e03/)  
3 - full internet support on non-eTCAM systems:  
[https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/](https://xrdocs.io/ncs5500/tutorials/2017-12-30-full-internet-view-on-base-ncs-5500-systems-s01e04/)   
4 - very large routing table in eTCAM systems:  
[https://xrdocs.io/ncs5500/tutorials/2018-01-25-s01e05-large-routing-tables-on-scale-ncs-5500-systems/](https://xrdocs.io/ncs5500/tutorials/2018-01-25-s01e05-large-routing-tables-on-scale-ncs-5500-systems/)  
5 - NCS5500 Jericho+ Systems and their Scalability:  
[https://xrdocs.io/ncs5500/tutorials/Understanding-ncs5500-jericho-plus-systems/](https://xrdocs.io/ncs5500/tutorials/Understanding-ncs5500-jericho-plus-systems/)  

In the third article, we demonstrated a feature used to optimize the route distribution between LEM and LPM to internet distribution. It was updated with a disclaimer in September 2019, we don’t recommend to use non-SE systems (Jericho or Jericho+) for full internet view. It will be different with Jericho2, but we will keep it for a dedicated post.  

We have now three remaining options:  
- Jericho / Qumran-MX with NL eTCAM
- Jericho+ with OP eTCAM
- Jericho+ with large LPM

## Let’s try to predict the future (with some really bad science)

Based on the routing table growth during the last 6 years, we can try to “project” the evolution of the table in the near future (let’s say, 9 years).  
From a pure scientific perspective, it is not worth much. I admit it. So let’s take it for what it is, a wet finger estimation.  
Let me explain the “method” used to get the number we will eventually use to estimate the resources utilization.

Sources: routes as seen in AS6447:  
[https://bgp.potaroo.net/as6447/](https://bgp.potaroo.net/as6447/)  
and  
[https://bgp.potaroo.net/v6/as6447/](https://bgp.potaroo.net/v6/as6447/)

Also, we used the following data:
- full table v4 per day:  
[https://bgp.potaroo.net/as6447/bgp-active.txt](https://bgp.potaroo.net/as6447/bgp-active.txt)  
- IPv4/24 routes per day:  
[https://bgp.potaroo.net/as6447/bgp-prefix-24.txt](https://bgp.potaroo.net/as6447/bgp-prefix-24.txt)  
- full table v6 per day:  
[https://bgp.potaroo.net/v6/as6447/bgp-active.txt](https://bgp.potaroo.net/v6/as6447/bgp-active.txt)  
- IPv6/48 routes per day:  
[https://bgp.potaroo.net/v6/as6447/bgp-prefix-48.txt](https://bgp.potaroo.net/v6/as6447/bgp-prefix-48.txt)

And to convert epoch / UNIX timestamps in human readable dates: h[ttps://www.epochconverter.com/](ttps://www.epochconverter.com/)  

I started writing the article the 10th of July, so I collected the figures from the sources above the 10th of July of each year.  

| Year | IPv4 total | IPv6 total | v4/24 | v6/48 |
|:---:|:---:|:---:|:---:|:---:|
| 2014 | 522,313 | 19,025 | 275,805 | 7,978 |
| 2015 | 587,105 | 24,651 | 310,949 | 10,607 |
| 2016 | 645,833 | 33,495 | 351,527 | 14,438 |
| 2017 | 710,976 | 43,249 | 390,413 | 18,727 |
| 2018 | 761,170 | 59,330 | 422,454 | 25,383 |
| 2019 | 817,799 | 74,613 | 464,433 | 34,508 |
| 2020 | 865,274 | 91,133 | 500,530 | 44,876 |

Let’s focus on the IPv4 part first, and more particularly to the /24 population.

| Year | IPv4 total | Growth | v4/24 | Growth | Growth Increase | non v4/24 | Growth | Growth Increase | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2014 | 522,313 | - | 275,805 | - | - | 246,508 | - | - |
| 2015 | 587,105 | 64,792 | 310,949 | 35,144 | - | 276,156 | 29,648 | - |
| 2016 | 645,833 | 58,728 | 351,527 | 40,578 | 5,434 | 294,306 | 18,150 | -11,498 |
| 2017 | 710,976 | 65,143 | 390,413 | 38,886 | -1,692 | 320,563 | 26,257 | 8,107 |
| 2018 | 761,170 | 50,194 | 422,454 | 32,041 | -6,845 | 338,716 | 18,153 | -8,104 |
| 2019 | 817,799 | 56,629 | 464,433 | 41,979 | 9,938 | 353,366 | 14,650 | -3,503 |
| 2020 | 865,274 | 47,475 | 500,530 | 36,097 | -5,882 | 364,744 | 11,378 | -3,272 |

The “Growth" is showing the difference between two sub-sequent years and the "Growth Increase" shows the difference between two sub-sequent growth rates. The second number should help identifying a linear or an algorithmic progression.

IPv4/24 growth trend: it doesn’t seem we have a clear trend here, between the different years we see various numbers scattered from -6,845 to +9,938.  

Conclusion: we will consider it’s a linear growth and we will estimate the number of new IPv4/24 per year: +42,000 prefixes.
{: .notice—info}


Non-IPv4 growth: except in 2017, it shows the growth is progressively slowing down.  

Conclusion: here it will be a totally arbitrary decision to continue this trend with lower and lower numbers (starting from -2,000 in 2021 to -200 in 2029).
{: .notice—info}

The results of this projection are the following:

| Year | IPv4 total | Growth | v4/24 | Growth | Growth Increase | non v4/24 | Growth | Growth Increase | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2019 | 817,799 | 56,629 | 464,433 | 41,979 | 9,938 | 353,366 | 14,650 | -3,503 |
| 2020 | 865,274 | 47,475 | 500,530 | 36,097 | -5,882 | 364,744 | 11,378 | -3,272 |
| 2021 | 919,274 | 54,000 | 542,530 | 42,000 | 5,903 | 376,744 | 9,378 | -2,000 | 
| 2022 | 969,152 | 49,878 | 584,530 | 42,000 | 0 | 384,622 | 7,878 | -1,500 | 
| 2023 | 1,018,030 | 48,878 | 626,530 | 42,000 | 0 | 391,500 | 6,878 | -1,000 | 
| 2024 | 1,066,108 | 48,078 | 668,530 | 42,000 | 0 | 397,578 | 6,078 | -800 | 
| 2025 | 1,113,586 | 47,478 | 710,530 | 42,000 | 0 | 403,056 | 5,478 | -600 | 
| 2026 | 1,160,564 | 46,978 | 752,530 | 42,000 | 0 | 408,034 | 4,978 | -500 | 
| 2027 | 1,207,142 | 46,578 | 794,530 | 42,000 | 0 | 412,612 | 4,578 | -400 | 
| 2028 | 1,253,420 | 46,278 | 836,530 | 42,000 | 0 | 416,890 | 4,278 | -300 | 
| 2029 | 1,299,498 | 46,078 | 878,530 | 42,000 | 0 | 420,968 | 4,078 | -200 | 

![IPv4-projection.png]({{site.baseurl}}/images/IPv4-projection.png){: .align-center}

Checking other sources on the web, like this APNIC article from Geoff Huston:  
[https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/](https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/)  
They predict 1,079,000 routes for Jan 2025, so it matches our “model”, predicting something between 1,066,108 and 1,113,586 IPv4 prefixes between July 2024 and July 2025.

Let’s study the IPv6 internet table evolution. Due to the smallest size of this table, it’s more hazardous to create a projection, but it won’t prevent me from making arbitrary asumptions to build my model ;)

| Year | IPv6 total | Growth | Growth Increase | v6/48 | Growth | Growth Increase | non v6/48 | Growth | Growth Increase | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2014 | 19,025 | - | - | 7,978 | - | - | 11,047 | - | - |
| 2015 | 24,651 | 5,626 | - | 10,607 | 2,629 | - |  14,044 | 2,997 | - | 
| 2016 | 33,495 | 8,844 | 3,218 | 14,438 | 3,831 | 1,202 | 19,057 | 5,013 | 2,016 | 
| 2017  | 43,249 | 9,754 | 910 | 18,727 | 4,289 | 458 | 24,522 | 5,465 | 452 | 
| 2018 | 59,330 | 16,081 | 6,327 | 25,383 | 6,656 | 2,367 | 33,947 | 9,425| 3,960 | 
| 2019 | 74,613 | 15,283 | -798 | 34,508 | 9,125 | 2,469 | 40,105 | 6,158 | -3,267 | 
| 2020 | 91,133 | 16,520 | 1,237 | 4,4876 | 10,368 | 1,243 | 46,257 | 6,152 | -6 | 

More particularly, we will pay attention to the /48 prefixes and their progression.

IPv6/48 growth trend: The "growth increase" varies from +458 to +2,469 during the last 5 years.  

Conclusion: Let’s take the highest number for the projection, rounded to +2500.
{: .notice—info}

Non-IPv6/48 growth trend: The “growth” varies from 2,997 to 9,425 but with a majority of the years around 6,000.  

Conclusion: For this one, we will take a totaly arbitrary growth of 6,000 new IPv6 prefixes (non /48) per year.
{: .notice—info}

The results of this projection are the following:

| Year | IPv6 total | Growth | Growth Increase | v6/48 | Growth | Growth Increase | non v6/48 | Growth | Growth Increase |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2019 | 74,613 | 15,283 | -798 | 34,508 | 9,125 | 2,469 | 40,105 | 6,158 | -3,267 | 
| 2020 | 91,133 | 16,520 | 1,237 | 44,876 | 10,368 | 1,243 | 46,257 | 6,152 | -6 | 
| 2021 | 110,001 | 18,868 | 2,348 | 57,744 | 12,868 | 2,500 | 52,257 | 6,000 | -152 | 
| 2022 | 131,369 | 21,368 | 2,500 | 73,112 | 15,368 | 2,500 | 58,257 | 6,000 | 0 | 
| 2023 | 155,237 | 23,868 | 2,500 | 90,980 | 17,868 | 2,500 | 64,257 | 6,000 | 0 | 
| 2024 | 181,605 | 26,368 | 2,500 | 111,348 | 20,368 | 2500 | 70,257 | 6,000 | 0 | 
| 2025 | 210,473 | 28,868 | 2,500 | 134,216 | 22,868 | 2,500 | 76,257 | 6,000 | 0 | 
| 2026 | 241,841 | 31,368 | 2 500 | 159,584 | 25,368 | 2,500 | 82,257 | 6,000 | 0 |  
| 2027 | 275,709 | 33,868 | 2,500 | 187,452 | 27,868 | 2,500 | 88,257 | 6,000 | 0 | 
| 2028 | 312,077 | 36,368 | 2,500 | 217,820 | 30,368 | 2,500 | 94,257 | 6,000 | 0 | 
| 2029 | 350,945 | 38,868 | 2,500 | 250,688 | 32,868  | 2,500 | 100,257 | 6,000 | 0 | 

![IPv6-projection.png]({{site.baseurl}}/images/IPv6-projection.png){: .align-center}

Here again we can compare this guesstimation with Geoff’s projection  
[https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/](https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/)  
In January 2025, they present a range starting from 160,000 for the linear model and 318,000 for the algorithmic one, while we plan for something between 181,605 and 210,473.  

Admittedly, it’s pretty vague ;)  

Let’s take these numbers nevertheless and see where these routes will be stored depending on the different products.

In summary, we project the following distribution:  

| Date (10th of July of each year) | IPv4 total | v4/24 | v4 non/24 | IPv6 total | v6/48 | v6 non/48 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2020 | 865,274 | 500,530 | 364,744 | 91,133 | 44,876 | 46,257 |
| 2021 | 919,274 | 542,530 | 376,744 | 110,001 | 57,744 | 52,257 |
| 2022 | 969,152 | 584,530| 384,622 | 131,369 | 73,112 | 58,257 |
| 2023 | 1,018,030 | 626,530 | 391,500 | 155,237 | 90,980 | 64,257 |
| 2024 | 1,066,108 | 668,530 | 397,578 | 181,605 | 111,348 | 70,257 |
| 2025 | 1,113,586 | 710,530 | 403,056 | 210,473 | 134,216 | 76,257 |
| 2026 | 1,160,564 | 752,530 | 408,034 | 241,841 | 159,584 | 82,257 |
| 2027 | 1,207,142 | 794,530 | 412,612 | 275,709 | 187,452 | 88,257 |
| 2028 | 1,253,420 | 836,530 | 416,890 | 312,077 | 217,820 | 94,257 |
| 2029 | 1,299,498 | 878,530 | 420,968 | 350,945 | 250,688 | 100,257 |

## Products and ASICs

In summary:

| Jericho / Qumran-MX with NL eTCAM | Jericho+ with OP eTCAM | Jericho+ with large LPM (and no eTCAM) |
|:---:|:---:|:---:|
| NCS5501-SE | NCS55A1-36H-SE-S | NCS55A1-24H |
| NCS5502-SE | NCS55A2-MOD-SE-S | NCS55A1-48Q6H |
| NC55-24X100G-SE | NC55-A-36X100-SE-S | NCS55A1-24H6H-SS |

### Jericho / Qumran-MX with NL eTCAM

Products:
- NCS5501-SE
- NCS5502-SE
- NC55-24X100G-SE

![JerichoNL12k.png]({{site.baseurl}}/images/JerichoNL12k.png){: .align-center}

### Jericho+ with OP eTCAM

Products:
- NCS55A1-36H-SE-S
- NCS55A2-MOD-SE-S
- NC55-A-36X100-SE-S

![Jplus-OP.png]({{site.baseurl}}/images/Jplus-OP.png){: .align-center}

### Jericho+ with large LPM (and no eTCAM)

Products:
- NCS55A1-24H
- NCS55A1-48Q6H
- NCS55A1-24H6H-SS

Case 1: Default configuration is Host Optimized

![LargeLPM-host-opt.png]({{site.baseurl}}/images/LargeLPM-host-opt.png){: .align-center}

Case 2: user changed to host-optimized-disable

![LargeLPM-host-disable.png]({{site.baseurl}}/images/LargeLPM-host-disable.png){: .align-center}

## Lab and Test

In this section, we will inject a real table in the routers and collect utilization statistics for the different resources (LEM, LPM and potentially external TCAM). Then, we will inject v4/24s, v6/48s and other routes following the estimated progression described above, and we will see the impact on resources, year after year.  

It certainly very fun (is it?) but the real purpose of this exercice is to extract valuable and actionable information out of these tests. For example, what hw-module profile we should use in the future depending on the ASIC type.

### Starting point

We use a public v4/v6 view collected in 2019. It shows:  
- 790,780 IPv4 prefixes
	- 445,773 /24s
    - 345,007 non-/24s
- 72,949 IPv6 prefixes
	- 35,009 /48s
    - 37,940 non-/48s

Jericho / Qumran-MX with NL eTCAM

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-2-702#sh bgp sum
BGP router identifier 1.3.5.8, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 4539177
BGP main routing table version 4539177
BGP NSR Initial initsync version 4 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         4539177    4539177    4539177    4539177     4539177           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.153   0   100 1719117      16  4539177    0    0 00:13:25     <mark>790771</mark>

RP/0/RP0/CPU0:5508-2-702#
RP/0/RP0/CPU0:5508-2-702#sh bgp ipv6 un sum
BGP router identifier 1.3.5.8, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 72953
BGP main routing table version 72953
BGP NSR Initial initsync version 4 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker           72953      72953      72953      72953       72953           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100   72965      16    72953    0    0 00:14:01      <mark>72948</mark>

RP/0/RP0/CPU0:5508-2-702#
RP/0/RP0/CPU0:5508-2-702#sh dpa resources iproute loc 0/7/CPU0

"iproute" OFA Table (Id: 41, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual            Capacity    Prefix   Actual            Capacity
 /0       26                20           /1       0                 20
 /2       4                 20           /3       7                 20
 /4       26                20           /5       0                 20
 /6       0                 20           /7       0                 20
 /8       11                20           /9       12                20
 /10      36                204          /11      97                409
 /12      285               818          /13      571               1636
 /14      1143              3275         /15      1913              5732
 /16      13184             42381        /17      7901              25387
 /18      13534             42585        /19      25210             86603
 /20      39182             127348       /21      47039             141679
 /22      100821            231968       /23      78898             207173
 <mark>/24      445838</mark>            1105590      /25      144               4299
 /26      211               4504         /27      383               3275
 /28      537               2866         /29      721               6961
 /30      3241              2866         /31      440               204
 /32      9657              20

OFA Infra Stats Summary
                 Create Requests: 1846483
                 Delete Requests: 1055411
                 Update Requests: 241726
                    Get Requests: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0

                          NPU ID: NPU-0                  NPU-1                  NPU-2                  NPU-3
           Create Server API Err: 0                      0                      0                      0
           Update Server API Err: 0                      0                      0                      0
           Delete Server API Err: 0                      0                      0                      0

RP/0/RP0/CPU0:5508-2-702#sh dpa resources ip6route loc 0/7/CPU0

"ip6route" OFA Table (Id: 42, Scope: Global)
--------------------------------------------------
IPv6 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       25           /1       0
 /2       0            /3       0
 /4       0            /5       0
 /6       0            /7       0
 /8       0            /9       0
 /10      25           /11      0
 /12      0            /13      0
 /14      0            /15      0
 /16      76           /17      0
 /18      0            /19      2
 /20      12           /21      4
 /22      7            /23      5
 /24      23           /25      7
 /26      13           /27      18
 /28      94           /29      2696
 /30      418          /31      179
 /32      12686        /33      1060
 /34      814          /35      517
 /36      2887         /37      501
 /38      908          /39      282
 /40      3689         /41      544
 /42      888          /43      144
 /44      4720         /45      465
 /46      2223         /47      1352
 <mark>/48      35009</mark>        /49      0
 /50      0            /51      1
 /52      0            /53      0
 /54      0            /55      0
 /56      219          /57      2
 /58      0            /59      0
 /60      16           /61      0
 /62      0            /63      3
 /64      87           /65      0
 /66      0            /67      0
 /68      0            /69      0
 /70      0            /71      0
 /72      0            /73      0
 /74      0            /75      0
 /76      0            /77      0
 /78      0            /79      0
 /80      0            /81      0
 /82      0            /83      0
 /84      0            /85      0
 /86      0            /87      0
 /88      0            /89      0
 /90      0            /91      0
 /92      0            /93      0
 /94      0            /95      0
 /96      0            /97      0
 /98      0            /99      0
 /100     0            /101     0
 /102     0            /103     0
 /104     25           /105     0
 /106     0            /107     0
 /108     0            /109     0
 /110     0            /111     0
 /112     0            /113     0
 /114     0            /115     0
 /116     0            /117     0
 /118     0            /119     0
 /120     2            /121     0
 /122     0            /123     0
 /124     1            /125     8
 /126     432          /127     16
 /128     97

OFA Infra Stats Summary
                 Create Requests: 73268
                 Delete Requests: 66
                 Update Requests: 0
                    Get Requests: 0

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0

                          NPU ID: NPU-0                  NPU-1                  NPU-2                  NPU-3
           Create Server API Err: 0                      0                      0                      0
           Update Server API Err: 0                      0                      0                      0
           Delete Server API Err: 0                      0                      0                      0

RP/0/RP0/CPU0:5508-2-702#</code>
</pre>
</div>

That places us somewhere between 2018 and 2019 on the estimation we built from the potaroo info.  We will start from 2019 and calculate what needs to be advertised on top of our full views.

A simple substraction will tell us how many routes "extra" we need to advertise to simulate the growth along the years:

| Year | v4/24 | Extra /24 | v4 non/24 | Extra others | v6/48 | Extra /48 | v6 non/48 | Extra others |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2020 | 500,530 | 54,757 | 364,744 | 19,737 | 44,876 | 9,867 | 46,257 | 8,317 |
| 2021 | 542,530 | 96,757 | 376,744 | 31,737 | 57,744 | 22,735 | 52,257 | 14,317 |
| 2022 | 584,530 | 138,757 | 384,622 | 39,615 | 73,112 | 38,103 | 58,257 | 20,317 |
| 2023 | 626,530 | 180,757 | 391,500 | 46,493 | 90,980 | 55,971 | 64,257 | 26,317 |
| 2024 | 668,530 | 222,757 | 397,578 | 52,571 | 111,348 | 76,339 | 70,257 | 32,317 | 
| 2025 | 710,530 | 264,757 | 403,056 | 58,049 | 134,216 | 99,207 | 76,257 | 38,317 |
| 2026 | 752,530 | 306,757 | 408,034 | 63,027 | 159,584 | 124,575 | 82,257 | 44,317 |
| 2027 | 794,530 | 348,757 | 412,612 | 67,605 | 187,452 | 152,443 | 88,257 | 50,317 | 
| 2028 | 836,530 | 390,757 | 416,890 | 71,883 | 217,820 | 182,811 | 94,257 | 56,317 |
| 2029 | 878,530 | 432,757 | 420,968 | 75,961 | 250,688 | 215,679 | 100,257 | 62,317 |

At the starting point we have:

**Jericho with NL eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5501-SE-6625#sh platf
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        NCS-5501-SE(Active)        IOS XR RUN        NSHUT
0/RP0/NPU0        Slice                      UP
0/FT0             NCS-1RU-FAN-FW             OPERATIONAL       NSHUT
0/PM1             NCS-1100W-ACFW             OPERATIONAL       NSHUT
RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 45013    (6 %)
        iproute                     : 10006    (1 %)
        ip6route                    : 35009    (4 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

RP/0/RP0/CPU0:5501-SE-6625#
RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 534746
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 38969    (7 %)
        iproute                     : 0        (0 %)
        ip6route                    : 39733    (7 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

RP/0/RP0/CPU0:5501-SE-6625#
RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources exttcamipv4 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv4

OOR Information
    NPU-0
        Estimated Max Entries       : 2048000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 781297   (38 %)
        iproute                     : 782062   (38 %)

RP/0/RP0/CPU0:5501-SE-6625#</code>
</pre>
</div>

**Jericho+ with OP eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-2-702#sh platf 0/0
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/0/CPU0          NC55-36X100G-A-SE          IOS XR RUN        NSHUT
RP/0/RP0/CPU0:5508-2-702#sh contr npu resources exttcamipv4 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv4
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 790992   (20 %)
        iproute                     : 791072   (20 %)

...
RP/0/RP0/CPU0:5508-2-702#sh contr npu resources exttcamipv6 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv6
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 73052    (4 %)
        ip6route                    : 73202    (4 %)

...
RP/0/RP0/CPU0:5508-2-702#</code>
</pre>
</div>

**Jericho+ with Large LPM and no eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh platf
Node              Type                       State             Config state
--------------------------------------------------------------------------------
0/RP0/CPU0        NCS-55A1-24H(Active)       IOS XR RUN        NSHUT
0/RP0/NPU0        Slice                      UP
0/RP0/NPU1        Slice                      UP
0/FT0             NC55-A1-FAN-FW             OPERATIONAL       NSHUT
0/FT1             NC55-A1-FAN-FW             OPERATIONAL       NSHUT
0/PM0             NCS-1100W-ACFW             FAILED            NSHUT
0/PM1             NCS-1100W-ACFW             OPERATIONAL       NSHUT
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 490276   (62 %)
        iproute                     : 455268   (58 %)
        ip6route                    : 35009    (4 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)
...
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1563508
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 373486   (24 %)
        iproute                     : 335526   (21 %)
        ip6route                    : 37956    (2 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)
...
RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

### Year 2020

We advertise the extra routes through a new peer (/24 + /23 for IPv4 and /48 + /47 for IPv6).  

On the route generator:

<div class="highlighter-rouge">
<pre class="highlight">
<code>pxe@pxe-ubuntu:~/routem$ more extra-v4-24H.2020
router bgp 100
bgp_id 192.168.100.153
neighbor 192.168.100.200 remote-as 100
neighbor 192.168.100.200 update-source 192.168.100.152
capability ipv4 unicast
capability refresh
<mark>network 1 11.1.1.0/24 54757</mark>
aspath 1 random 5
locpref 1 120
metric 1 5
<mark>network 2 51.1.1.0/23 19737</mark>
aspath 2 random 5
locpref 2 110
metric 2 10
sendall

pxe@pxe-ubuntu:~/routem$ more extra-v6-24H.2020
router bgp 152
bgp_id 192.168.100.11
neighbor 2001:111::200 remote-as 100
neighbor 2001:111::200 update-source 2001:111::152
capability ipv6 unicast
capability refresh
<mark>network 1 100:1:1:1/48 9867</mark>
aspath 1 random 5
locpref 1 120
metric 1 5
<mark>network 2 102:1:1:1/47 8317</mark>
aspath 2 random 5
locpref 2 110
metric 2 15
sendall
pxe@pxe-ubuntu:~/routem</code>
</pre>
</div>

And on the IOS XR router:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh bgp sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 1735636
BGP main routing table version 1735636
BGP NSR Initial initsync version 6 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         1735636    1735636    1735636    1735636     1735636           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100 1721352   12009  1735636    0    0    1d13h     790771
192.168.100.152   0   100     163      12  1735636    0    0 00:01:28      <mark>74494</mark>

RP/0/RP0/CPU0:24H-1-701#sh bgp ipv6 un sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 273405
BGP main routing table version 273405
BGP NSR Initial initsync version 6 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          273405     273405     273405     273405      273405           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100  148151   12245   273405    0    0 21:04:39      72948
2001:111::152     0   152     110     128   273405    0    0 00:00:07      <mark>18184</mark>

RP/0/RP0/CPU0:24H-1-701#sh dpa resources iproute loc 0/0/CPU0 | i /24
 /24      500530       /25      144
RP/0/RP0/CPU0:24H-1-701#sh dpa resources ip6route loc 0/0/CPU0 | i /48
 /48      44876        /49      0
RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

**Jericho with NL eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem

OOR Information
    NPU-0
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 54881    (7 %)
        iproute                     : 10007    (1 %)
        ip6route                    : 44876    (6 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm

OOR Information
    NPU-0
        Estimated Max Entries       : 549919
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 47287    (9 %)
        iproute                     : 0        (0 %)
        ip6route                    : 48051    (9 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

RP/0/RP0/CPU0:5501-SE-6625#sh contr npu resources exttcamipv4 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv4

OOR Information
    NPU-0
        Estimated Max Entries       : 2048000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green

Current Usage
    NPU-0
        Total In-Use                : 855743   (42 %)
        iproute                     : 856508   (42 %)

RP/0/RP0/CPU0:5501-SE-6625#</code>
</pre>
</div>

**Jericho+ with OP eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:5508-2-702#sh contr npu resources exttcamipv4 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv4
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 4000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 865438   (22 %)
        iproute                     : 865518   (22 %)
...
RP/0/RP0/CPU0:5508-2-702#sh contr npu resources exttcamipv6 loc 0/0/CPU0
HW Resource Information
    Name                            : ext_tcam_ipv6
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 2000000
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 91237    (5 %)
        ip6route                    : 91387    (5 %)

...</code>
</pre>
</div>

**Jericho+ with Large LPM and no eTCAM**

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 554898   (71 %)
        iproute                     : 510026   (65 %)
        ip6route                    : 44876    (6 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

...
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1559724
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 401492   (26 %)
        iproute                     : 355214   (23 %)
        ip6route                    : 46274    (3 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)

...
RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

### Year 2021 to year 2029

We continue to increase the amount of "extra" routes step by step and we gather the results in following charts.

**Jericho w/ NL12K eTCAM**

| Year | LEM Max | LEM in-use | LPM Max | LPM in-use | eTCAM Max | eTCAM in-use |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Starting Point | 786432 | 45013 | 534746 | 38969 | 2048000 | 781297 |
| 2020 | 786432 | 54881 | 549919 | 47287 | 2048000 | 855743 |
| 2021 | 786432 | 67749 | 554073 | 53287 | 2048000 | 908919 |
| 2022 | 786432 | 83113 | 558189 | 59287 | 2048000 | 957666 |
| 2023 | 786432 | 100985 | 562075 | 65287 | 2048000 | 1006238 |
| 2024 | 786432 | 121353 | 566489 | 71287 | 2048000 | 1053379 |
| 2025 | 786432 | 144221 | 564116 | 77287 | 2048000 | 1099520 |
| 2026 | 786432 | 169581 | 568307 | 83287 | 2048000 | 1146447 |
| 2027 | 786432 | 197449 | 565508 | 89287 | 2048000 | 1193007 |
| 2028 | 786432 | 227817 | 569891 | 95287 | 2048000 | 1239282 |

![LPM-Jericho-NL12k.png]({{site.baseurl}}/images/LPM-Jericho-NL12k.png){: .align-center}

![LEM-Jericho-NL12k.png]({{site.baseurl}}/images/LEM-Jericho-NL12k.png){: .align-center}

![eTCAM-Jericho-NL12k.png]({{site.baseurl}}/images/eTCAM-Jericho-NL12k.png){: .align-center}

Conclusion: these devices can handle the internet growth with no concern or limitation.
{: .notice—info}

**Jericho+ w/ OP eTCAM**

| Year | eTCAM Max | v4 in-use | v6 in-use |
|:---:|:---:|:---:|:---:|
| Starting Point | 4M+2M | 790992 | 73052 |
| 2020 | 4M+2M | 865438 | 91237 |
| 2021 | 4M+2M | 918614 | 110105 |
| 2022 | 4M+2M | 967361 | 131473 |
| 2023 | 4M+2M | 1015933 | 155341 |
| 2024 | 4M+2M | 1063074 | 181709 |
| 2025 | 4M+2M | 1109215 | 210577 |
| 2026 | 4M+2M | 1156142 | 241945 |
| 2027 | 4M+2M | 1202702 | 275813 |
| 2028 | 4M+2M | 1248977 | 312181 |

![Jplus OP.png]({{site.baseurl}}/images/Jplus OP.png){: .align-center}

Conclusion: these devices can handle the internet growth with zero concern or limitation, we have a lot of available space in the OP eTCAM.
{: .notice—info}

**Jericho+ with Large LPM and host-optimized (default mode)**

| Year | LEM Max | LEM in-use | LPM Max | LPM in-use |
|:---:|:---:|:---:|:---:|:---:|
| Starting Point | 786432 | 490276 | 1563508 | 373486 |
| 2020 | 786432 | 554898 | 1559724 | 401492 |
| 2021 | 786432 | 608960 | 1575645 | 419475 |
| 2022 | 786432 | 665324 | 1573918 | 433222 |
| 2023 | 786432 | 724944 | 1572299 | 446038 |
| 2024 | 786432 | 781632 | 1570948 | 458084 |
| 2025 | - | - | - | - |

![LPM-LargeLPM-hostOptEnable.png]({{site.baseurl}}/images/LPM-LargeLPM-hostOptEnable.png){: .align-center}

![LEM-LargeLPM-hostOptEnable.png]({{site.baseurl}}/images/LEM-LargeLPM-hostOptEnable.png){: .align-center}

We will cover what's happening in 2024 in the next section.

**Jericho+ with Large LPM and host-optimized-disable**

| Year | LEM Max | LEM in-use | LPM Max | LPM in-use |
|:---:|:---:|:---:|:---:|:---:|
| Starting Point | 786432 | 44509 | 1349849 | 819259 |
| 2020 | 786432 | 54373 | 1355608 | 902022 |
| 2021 | 786432 | 67241 | 1386730 | 961198 |
| 2022 | 786432 | 82609 | 1418157 | 1015945 |
| 2023 | 786432 | 100473 | 1422956 | 1070517 |
| 2024 | 786432 | 120841 | 1423107 | 1123656 |
| 2025 | 786432 | 143705 | 1412551 | 1175799 |
| 2026 | 786432 | 169077 | 1400564 | 1228726 |
| 2027 | 786432 | 196945 | 1388103 | 1281286 |
| 2028 | 786432 | 227313 | 1373880 | 1333561 |

![LPM-LargeLPM-hostOptDisable.png]({{site.baseurl}}/images/LPM-LargeLPM-hostOptDisable.png){: .align-center}

![LEM-LargeLPM-hostOptDisable.png]({{site.baseurl}}/images/LEM-LargeLPM-hostOptDisable.png){: .align-center}

### Special case of the Jericho+ with Large LPM

When simulating year 2024, we hit the first bottleneck with the J+ with large LPM systems:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh bgp sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 3153690
BGP main routing table version 3153690
BGP NSR Initial initsync version 6 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         3153690    3153690    3153690    3153690     3153690           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100 1721534   12191  3153690    0    0    1d16h     790771
192.168.100.152   0   100    1118     155  3153690    0    0 00:00:14     <mark>275328</mark>

RP/0/RP0/CPU0:24H-1-701#
RP/0/RP0/CPU0:24H-1-701#sh bgp ipv6 un sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 755765
BGP main routing table version 755765
BGP NSR Initial initsync version 6 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker          755765     755765     755765     755765      755765           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100  148333   14655   755765    0    0    1d00h      72948
2001:111::152     0   152    1063     764   755765    0    0 00:00:33     <mark>108656</mark>

RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Jul.27 08:23:12 PDT


Current Usage
        Total In-Use                : <mark>781632</mark>   <mark>(99 %)</mark>
        iproute                     : 670293   (85 %)
        ip6route                    : 111348   (14 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Jul.27 08:23:12 PDT


Current Usage
        Total In-Use                : 781632   (99 %)
        iproute                     : 670293   (85 %)
        ip6route                    : 111348   (14 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)


RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1570948
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 458084   (29 %)
        iproute                     : 387806   (25 %)
        ip6route                    : 70274    (4 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


NPU-1
OOR Summary
        Estimated Max Entries       : 1570948
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 458084   (29 %)
        iproute                     : 387806   (25 %)
        ip6route                    : 70274    (4 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


RP/0/RP0/CPU0:24H-1-701#
RP/0/RP0/CPU0:24H-1-701#sh dpa resources iproute loc 0/0/CPU0

"iproute" OFA Table (Id: 37, Scope: Global)
--------------------------------------------------
IPv4 Prefix len distribution
Prefix   Actual       Prefix   Actual
 /0       1            /1       0
 /2       4            /3       7
 /4       1            /5       0
 /6       0            /7       0
 /8       10           /9       12
 /10      36           /11      97
 /12      285          /13      571
 /14      1143         /15      1913
 /16      13184        /17      7901
 /18      13534        /19      25210
 /20      39182        /21      47039
 /22      100821       /23      131178
 /24      665574       /25      144
 /26      211          /27      383
 /28      537          /29      721
 /30      3241         /31      440
 /32      9496

OFA Infra Stats Summary
                 Create Requests: 1740765
                 Delete Requests: 677889
                 Update Requests: 13819

                          Errors
                Resolve Failures: 0
                 Not Found in DB: 0
                    Exists in DB: 0
                 No Memory in DB: 0
               Reserve Resources: 0
               Release Resources: 0
                Update Resources: 0

                          NPU ID: NPU-0                  NPU-1
          <mark> Create Server API Err: 4777                   4777</mark>
           Update Server API Err: 0                      0
           Delete Server API Err: 0                      0

RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

Starting from this point, the LEM is saturated while LPM is only used 29%.  
It would be profitable to disable the default "host-optimized" mode. With this configuration (requiring a reload of the product), we store the IPv4/24 routes in LPM, which is the largest memory of the system.

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#conf
RP/0/RP0/CPU0:24H-1-701(config)#hw-module fib ?
  dlb   Destination Based Load balancing
  ipv4  Configure ipv4 protocol
  ipv6  Configure ipv6 protocol
  mpls  Configure mpls protocol
RP/0/RP0/CPU0:24H-1-701(config)#hw-module fib ipv4 ?
  scale  Configure scale mode for no-TCAM card
RP/0/RP0/CPU0:24H-1-701(config)#hw-module fib ipv4 scale ?
  host-optimized-disable  Configure Host optimization by default
  internet-optimized      Configure Intetrnet optimized
RP/0/RP0/CPU0:24H-1-701(config)#hw-module fib ipv4 scale host-optimized-disable
In order to activate this new scale, you must manually reload the chassis/all line cards
RP/0/RP0/CPU0:24H-1-701(config)#commit
RP/0/RP0/CPU0:24H-1-701(config)#end
RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

After reloading the router, we verify LEM and LPM:

<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0

HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 120841   (15 %)
        iproute                     : 9496     (1 %)
        ip6route                    : 111348   (14 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

...
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lp

HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1423107
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 1123656  (79 %)
        iproute                     : 1053378  (74 %)
        ip6route                    : 70275    (5 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)</code>
</pre>
</div>

LEM is now very lightly used (15%) and we will use massively the LPM for most of our prefixes. That's why it's now loaded at 79% but keep in mind it's the largest memory of the system.

We continue advertising more and more routes and in 2028, we are getting very close to the limit for this chipset:
  
<div class="highlighter-rouge">
<pre class="highlight">
<code>RP/0/RP0/CPU0:24H-1-701#sh bgp sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0000000   RD version: 5960575
BGP main routing table version 5960575
BGP NSR Initial initsync version 629974 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         5960575    5960575    5960575    5960575     5960575           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
192.168.100.151   0   100 1719200      99  5960575    0    0 01:36:51     790771
192.168.100.152   0   100    2525      63  5960575    0    0 00:00:14     <mark>462640</mark>

RP/0/RP0/CPU0:24H-1-701#sh bgp ipv6 un sum
BGP router identifier 1.3.5.9, local AS number 100
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0xe0800000   RD version: 1939637
BGP main routing table version 1939637
BGP NSR Initial initsync version 181612 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

BGP is operating in STANDALONE mode.


Process       RcvTblVer   bRIB/RIB   LabelVer  ImportVer  SendTblVer  StandbyVer
Speaker         1939637    1939637    1939637    1939637     1939637           0

Neighbor        Spk    AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down  St/PfxRcd
2001:111::151     0   100   73048    8961  1939637    0    0 01:37:08      72949
2001:111::152     0   152    3031    1169  1939637    0    0 00:00:41     <mark>239128</mark>

RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lem loc 0/0/CPU0
HW Resource Information
    Name                            : lem
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 786432
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Green


Current Usage
        Total In-Use                : 227313   (29 %)
        iproute                     : 9496     (1 %)
        ip6route                    : 217820   (28 %)
        mplslabel                   : 0        (0 %)
        l2brmac                     : 0        (0 %)

...
RP/0/RP0/CPU0:24H-1-701#sh contr npu resources lpm loc 0/0/CPU0
HW Resource Information
    Name                            : lpm
    Asic Type                       : Jericho Plus

NPU-0
OOR Summary
        Estimated Max Entries       : 1373880
        Red Threshold               : 95
        Yellow Threshold            : 80
        OOR State                   : Red
        OOR State Change Time       : 2020.Jul.27 10:16:25 PDT


Current Usage
        Total In-Use                : <mark>1333561</mark>  <mark>(97 %)</mark>
        iproute                     : 1239283  (90 %)
        ip6route                    : 94275    (7 %)
        ipmcroute                   : 1        (0 %)
        ip6mcroute                  : 0        (0 %)
        ip6mc_comp_grp              : 0        (0 %)


...
RP/0/RP0/CPU0:24H-1-701#</code>
</pre>
</div>

Conclusion: it's advisable to disable the default profile "host-optimized", it will extend significatively the router capability when used with full internet view.
{: .notice—info}
  
# Conclusion
  
In this post, we created a prevision model for the internet size progression.  
No doubt it can be refined. For example, we only considered the "non-IPv4/24 routes" and "non-IPv6/48 routes" as a block. Also because we advertised only subsequent /23s and /47s to create the "extra prefixes". Much better can be done here (hopefully by someone else ;)).  
Take all this with a grain of salt. Also, if you are aware of more precise prediction models (with an evolution for individual prefix length), please let us know.
In the lab, we simulated internet routing from 2020 to 2028 and we examined each memory consumption (LEM, LPM and when present, eTCAM).  
The systems with external TCAM are clearly showing a ton of free space for internet, even in 8 years.  
The systems based on Jericho+ with large LPM can also be used for internet peering, but it may be required to disable the "host-optimized" mode in a couple of years (around 2024) to leverage the large size of the LPM and offer 4+ more years of growth.

Next episode: Jericho2. Stay tuned.
