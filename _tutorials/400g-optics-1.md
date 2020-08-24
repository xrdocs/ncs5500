---
published: true
date: '2020-06-16 19:18 -0700'
title: NCS-5700 400G Optics Technology and Roadmap
author: Vincent Ng
excerpt: NCS-5700 400G Optics Support
tags:
  - iosxr
  - NCS-5700
  - 400G
  - Optics
position: hidden
---
## NCS-5700 400G Optics Technology and Roadmap

NCS-5700 is the first platform in the NCS-5500 Series that supports 400G optics, which is introduced in a previous tutorial:

[Introducing 400GE on NCS5500 Series](https://xrdocs.io/ncs5500/tutorials/introducing-400ge-on-ncs5500-series/)

This tutorial will discuss in more details the 400G optics technology and roadmap for the NCS-5700 series.

A follow on tutorial will discuss the 400G optics and breakout options supported for each port type on the NCS-5700 series linecards, NC57-24DD and NC57-18DD-SE.


## Introducing QSFP-DD MSA

QSFP-DD is the leading form factor for 400G optics, promoted by QSFP-DD [MSA](http://www.qsfp-dd.com/). Members include Cisco and other major suppliers.

QSFP+ and QSFP28 are the de-facto standard for high density 40G and 100G optics.

QSFP-DD is fully compatible with QSFP+ and QSFP28, and so maintains the same port density.

The QSFP-DD MSA published its Revision 1.0 specifications in Sep 2016, and it is at its Revision 5.0 as of Jul 2019.

<img src="{{site.baseurl}}/images/400G/qdd-q28.png" width="450" />

### Number of Serdes

![]({{site.baseurl}}/images/400G/qdd-2-row.png){: .align-right}
QSFP-DD adds a second row of pins and increases the number of serdes from 4 to 8, hence the name double-density.

Therefore it supports optics up to 8 electrical lanes.

### 50G Serdes Support

QSFP-DD increases the maximum serdes speed from 25G to 50G, therefore supporting a maximum speed of 8x50G = 400G aggregate.

For backward compatibility, it also supports 10G and 25G serdes speed, thus allowsing all possible aggregate speeds of 40G, 100G, 200G and 400G.

Optics of all form factors, such as QSFP+, QSFP28, QSFP56, QSFP28-DD and QSFP56-DD are supported by QSFP-DD.

### PAM4 Encoding Support

QSFP-DD supports PAM4 encoding which is used by 50G serdes, while maintaining the use of NRZ encoding for 10G/25G serdes.

### RS-FEC RS(544,514) Support

QSFP-DD supports RS-FEC (Clause 91) RS(544,514) for 50G serdes, while maintaining the use of RS(528,514) for 25G serdes.

### High Power Support

QSFP-DD supports high power optics, therefore ready for coherent optics such as 400G-ZR and 400G-ZR+.

At OFC 2019, Cisco Demonstrate 20W+ power dissipation of QSFP-DD.

[https://blogs.cisco.com/sp/cisco-demonstrates-20w-power-dissipation-of-qsfp-dd-at-ofc-2019](https://blogs.cisco.com/sp/cisco-demonstrates-20w-power-dissipation-of-qsfp-dd-at-ofc-2019)

[https://www.lightwaveonline.com/optical-tech/transmission/article/14036073/qsfpdd-msa-releases-common-management-interface-40-and-hardware-specification-50](https://www.lightwaveonline.com/optical-tech/transmission/article/14036073/qsfpdd-msa-releases-common-management-interface-40-and-hardware-specification-50)

### Flexible Breakout Support

The wide range of combination of serdes number and speeds provides very flexible breakout options:

<style>
table {
  width: auto;
}
</style>
 &nbsp; &nbsp; QSFP56-DD &nbsp; &nbsp; | &nbsp; &nbsp; QSFP28-DD &nbsp; &nbsp; | &nbsp; &nbsp; QSFP56 &nbsp; &nbsp; | &nbsp; &nbsp; QSFP28 &nbsp; &nbsp; | &nbsp; &nbsp; QSFP+ &nbsp; &nbsp;
:---------:|:---------:|:------:|:------:|:-----:
 400G      | 200G      | 200G   | 100G   | 40G   
 2x200G    | 2x100G    | 2x100G | 4x25G  | 4x10G 
 4x100G    | 8x25G     | 4x50G  |        |       
 8x50G     |           |        |        |       

## 400G Optics Technology Evolution

For early QSFP28 100G optics, majority are using 4x 25G wavelengths with NRZ encoding.

With evolution to 200G/400G optics, there is a need for higher speed wavelengths such as 50G/100G in order to reduce lasers cost and complexity.

25G wavelengths are supported using NRZ encoding with 25GBaud and RS-FEC RS(528,514). 16x 25G wavelengths are required for an aggregate 400G speed.

50G wavelengths are supported using PAM4 encoding with 25GBaud and RS-FEC RS(544,514). 8x 50G wavelengths are required for an aggregate 400G speed.

100G wavelengths are supported using PAM4 encoding with 50GBaud and RS-FEC RS(544,514). An even lower number of wavelengths, 4x, are required for an aggregate 400G speed.

For short distnaces up to about 100m, usually multimode fibers (MMF) are deployed for lower cost. Furthermore, parallel fibers, each with single wavelength, are used to reduce optics complexity. 

Parallel MMF has additional advantage of supporting breakout from 400G to multiple lower speed optics for more flexible deployment.

For medium distances to about 500m, usually parallel single mode fibers (SMF) are deployed. This could also support breakout.

For longer distances 2km and beyong, usually a single duplex SMF fiber pair is deployed to minimize fiber numbers and cost. Multiple wavelengths will be multiplexed into a single SMF using WDM technologies, such as CWDM or LWDM. LWDM is higher cost and usually for longer distances.

To reach even longer distances like 80km and above, usually Coherent Detection optics are used, which have their own specific encoding and FEC, and have single tunable wavelength at speed 100G or 400G. These wavelengths may even be transported over long haul DWDM systems and reach 1000's of km's.

This is a brief summary of the latest Optics standards with 50G, 100G or 400G wavelengths:

Date     | Specs         | Stardard       | Speed | W/L  | Distance | Cable     | Type     | Breakout | WDM
---------|---------------|----------------|-------|------|----------|-----------|----------|----------|---
2017 Dec | IEEE 802.3bs  | 400GBase-SR16  | 400G  | 25G  | 100m     | MMF OM4   | Parallel | Y        |
         |               | 400GBase-DR4   | 400G  | 100G | 500m     | SMF       | Parallel | Y        |
         |               | 400GBase-FR8   | 400G  | 50G  | 2km      | SMF       | Duplex   | N        | LWDM
         |               | 400GBase-LR8   | 400G  | 50G  | 10km     | SMF       | Duplex   | N        | LWDM
         |               | 200GBase-DR4   | 200G  | 50G  | 500m     | SMF       | Parallel | Y        |
         |               | 200GBase-FR4   | 200G  | 50G  | 2km      | SMF       | Duplex   | N        | CWDM
         |               | 200GBase-LR4   | 200G  | 50G  | 10km     | SMF       | Duplex   | N        | LWDM
2018 Jun | MSA [ETC](https://ethernettechnologyconsortium.org/) | 400GBase-KR8   | 400G  | 50G  | 1m       | Backplane | Parallel | Y        |
         |               | 400GBase-CR8   | 400G  | 50G  | 5m       | Copper    | Parallel | Y        |
2018 Sep | MSA [100G LD](http://100glambda.com/) | 100G-FR        | 100G  | 100G | 2km      | SMF       | Duplex   | N        |
         |               | 100G-LR        | 100G  | 100G | 10km     | SMF       | Duplex   | N        |
         |               | 400G-FR4       | 400G  | 100G | 2km      | SMF       | Duplex   | N        | CWDM
         |               | 400G-LR4-10    | 400G  | 100G | 10km     | SMF       | Duplex   | N        | CWDM
2018 Dec | IEEE 802.3cd  | 200GBase-KR4   | 200G  | 50G  | 1m       | Backplane | Parallel | Y        |
         |               | 200GBase-CR4   | 200G  | 50G  | 5m       | Copper    | Parallel | Y        |
         |               | 200GBase-SR4   | 200G  | 50G  | 100m     | MMF OM4   | Parallel | Y        |
         |               | 100GBase-KR2   | 100G  | 50G  | 1m       | Backplane | Parallel | Y        |
         |               | 100GBase-CR2   | 100G  | 50G  | 5m       | Copper    | Parallel | Y        |
         |               | 100GBase-SR2   | 100G  | 50G  | 100m     | MMF OM4   | Parallel | Y        |
         |               | 100GBase-DR    | 100G  | 100G | 500m     | SMF       | Duplex   | N        |
2019 Mar | MSA [SWDM](http://www.swdm.org/)     | 100G-SWDM2     | 100G  | 50G  | 100m     | MMF OM4   | Duplex   | N        | Bidi
2019 Nov | IEEE 802.3cn  | 400GBase-ER8   | 400G  | 50G  | 40km     | SMF       | Duplex   | N        | LWDM
         |               | 200GBase-ER4   | 200G  | 50G  | 40km     | SMF       | Duplex   | N        | LWDM
2020 Jan | IEEE 802.3cm  | 400GBase-SR8   | 400G  | 50G  | 100m     | MMF OM4   | Parallel | Y        |
         |               | 400GBase-SR4.2 | 400G  | 50G  | 100m     | MMF OM4   | Parallel | Y        | Bidi
2020 Mar | MSA [OIF](https://www.oiforum.com/technical-work/hot-topics/400zr-2/) | 400ZR          | 400G  | 400G | <120km   | SMF       | Duplex   | N        | Coherent
2020 End | IEEE P802.3cu | 400GBase-FR4   | 400G  | 100G | 2km      | SMF       | Duplex   | N        | CWDM
         |               | 400GBase-LR4-6 | 400G  | 100G | 6km      | SMF       | Duplex   | N        | CWDM
2020 End | MSA [OpenZR+](http://openzrplus.org/) | 400G-ZR+       | 400G  | 400G | >120km   | SMF       | Duplex   | N        | Coherent
2021 End | IEEE P802.3ck | 400GBase-CR4   | 400G  | 100G | 5m       | Copper    | Parallel | Y        |
         |               | 200GBase-CR2   | 200G  | 100G | 5m       | Copper    | Parallel | Y        |
         |               | 100GBase-CR    | 100G  | 100G | 5m       | Copper    | Duplex   | N        |
2021 End | IEEE P802.3ct | 100GBase-ZR    | 100G  | 100G | 80km     | SMF       | Duplex   | N        | Coherent
2022 Mid | IEEE P802.3cw | 400GBase-ZR    | 400G  | 400G | 80km     | SMF       | Duplex   | N        | Coherent
2022 Mid | IEEE P802.3db | 400GBase-SR4   | 400G  | 100G | 50m      | MMF OM4   | Parallel | Y        |
         |               | 200GBase-SR2   | 200G  | 100G | 50m      | MMF OM4   | Parallel | y        |
         |               | 100GBase-SR    | 100G  | 100G | 50m      | MMF OM4   | Duplex   | N        |

### NCS-5700 QSFP-DD 400G Optics with 50G/100G PAM4 Wavelengths Support and Roadmap

400G optics support will be introduced in phases for NCS-5700.

This section will cover all 400G optics with new gneration 50G and 100G wavelengths, and using QSFP-DD form factor.

For ease of visualising the use case of each optics, we have organized the optics into three major catagories, each with varying distances and breakout capabilities.

- Cables: Usually comes in passive copper Direct Attach Cables (DAC) for a few meters, or active optical cables (AOC) up to 30m. It could be direct, or breakout like 8x50 or 4x100.

- Multimode Fibers: Usually Parallel MMF up to about 100m for OM4, and capable of breakout.

- Single mode Fibers: Could be parallsl SMF or duplex SMF. Parallel SMF is cable of breakout, and Duplex SMF will need WDM for multiplexing multiple wavelengths into a single SMF.











