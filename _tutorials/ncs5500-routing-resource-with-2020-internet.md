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
position: hidden
---
{% include toc icon="table" title="NCS5500 and the 2020 Internet" %} 

![NCS 5500 INTERNAL RESOURCES.png]({{site.baseurl}}/images/NCS 5500 INTERNAL RESOURCES.png){: .align-center}

You can find more content related to NCS5500 including routing memory management, VRF, URPF, Netflow, QoS, EVPN, Flowspec implementation following this [link](https://xrdocs.io/ncs5500/tutorials/).

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

Non-IPv4 growth: except in 2017, it shows the growth is progressively slowing down.  
Conclusion: here it will be a totally arbitrary decision to continue this trend with lower and lower numbers (starting from -2,000 in 2021 to -200 in 2029).

The results of this projection are the following:

| Year | IPv4 total | Growth | v4/24 | Growth | Growth Increase | non v4/24 | Growth | Growth Increase | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2019 | 817,799 | 56,629 | 464,433 | 41,979 | 9,938 | 353,366 | 14,650 | -3,503 |
| 2020 | 86,5274 | 4,7475 | 500,530 | 36,097 | -5,882 | 364,744 | 11,378 | -3,272 |
| 2021 | 91,9274 | 54,000 | 542,530 | 4,2000 | 5,903 | 376,744 | 9,378 | -2,000 | 
| 2022 | 969,152 | 49,878 | 584,530 | 42,000 | 0 | 384,622 | 7,878 | -1,500 | 
| 2023 | 1,018,030 | 48,878 | 626,530 | 42,000 | 0 | 391,500 | 6,878 | -1,000 | 
| 2024 | 1,066,108 | 48,078 | 668,530 | 42,000 | 0 | 397,578 | 6,078 | -800 | 
| 2025 | 1,113,586 | 47,478 | 710,530 | 42,000 | 0 | 403,056 | 5,478 | -600 | 
| 2026 | 1,160,564 | 46,978 | 752,530 | 42,000 | 0 | 408,034 | 4,978 | -500 | 
| 2027 | 1,207,142 | 46,578 | 794,530 | 42,000 | 0 | 412,612 | 4,578 | -400 | 
| 2028 | 1,253,420 | 46,278 | 836,530 | 42,000 | 0 | 416,890 | 4,278 | -300 | 
| 2029 | 1,299,498 | 46,078 | 878,530 | 42,000 | 0 | 420,968 | 4,078 | -200 | 

Checking other sources on the web, like this APNIC article from Geoff Huston:  
[https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/](https://blog.apnic.net/2020/01/14/bgp-in-2019-the-bgp-table/)  
They predict 1,079,000 routes for Jan 2025, so it matches our “model”, predicting something between 1,066,108 and 1,113,586 IPv4 prefixes between July 2024 and July 2025.

Let’s study the IPv6 internet table evolution. Due to the smallest size of this table, it’s more hazardous to create projection, but it won’t prevent me from making the wildest guess to b

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

Non IPv6/48 growth trend: The “growth” varies from 2,997 to 9,425 but with a majority of the years around 6,000.  
Conclusion: For this one, we will take a totaly arbitrary growth of 6,000 new IPv6 prefixes (non /48) per year.

The results of this projection are the following:

| Year | IPv6 total | Growth | v6/48 | Growth | Growth Increase | non v6/48 | Growth | Growth Increase | 
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 2019 | 74,613 | 15,283 | -798 | 34,508 | 9,125 | 2,469 | 40,105 | 6,158 | -3,267 | 
| 2020 | 91,133 | 16,520 | 1,237 | 4,4876 | 10,368 | 1,243 | 46,257 | 6,152 | -6 | 
| 2021 | 110,001 | 18,868 | 2,348 | 57,744 | 12,868 | 2,500 | 52,257 | 6,000 | -152 | 
| 2022 | 131,369 | 21,368 | 2,500 | 73,112 | 15,368 | 2,500 | 58,257 | 6,000 | 0 | 
| 2023 | 155,237 | 23,868 | 2,500 | 90,980 | 17,868 | 2,500 | 64,257 | 6,000 | 0 | 
| 2024 | 181,605 | 26,368 | 2,500 | 111,348 | 20,368 | 2500 | 70,257 | 6,000 | 0 | 
| 2025 | 210,473 | 28,868 | 2,500 | 134,216 | 22,868 | 2,500 | 76,257 | 6,000 | 0 | 
| 2026 | 241,841 | 31,368 | 2 500 | 159,584 | 25,368 | 2,500 | 82,257 | 6,000 | 0 |  
| 2027 | 275,709 | 33,868 | 2,500 | 187,452 | 27,868 | 2,500 | 88,257 | 6,000 | 0 | 
| 2028 | 312,077 | 36,368 | 2,500 | 217,820 | 30,368 | 2,500 | 94,257 | 6,000 | 0 | 
| 2029 | 350,945 | 38,868 | 2,500 | 250,688 | 32,868  | 2,500 | 100,257 | 6,000 | 0 | 

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

