---
published: true
date: '2023-06-23 13:28 -0700'
title: Filtering IPv6 Packets with Extension Headers on NCS5500/5700
author: Tejas Lad and Bala Murali Krishna Sanka
excerpt: >-
  This document will show the NCS5500/5700 capability to filter IPv6 packets
  with extension headers
tags:
  - iosxr
  - cisco
  - NCS5500
  - NCS5700
  - IPv6
  - ACL
  - IPv6 EH
postion: top
---
{% include toc icon="table" title="Table of Contents" %}

|Tejas Lad, Technical Marketing Engineer|  
|Bala Murali Krishna Sanka, Technical Marketing Engineer|

## Introduction

In our previous [tech-note](https://xrdocs.io/ncs5500/tutorials/user-defined-field-ncs55xx-and-ncs5xx/), we saw how User Defined Fields - UDF can provide advanced filtering capabilities on NCS5500/5700. We saw examples of how different fields can be matched and used to filter traffic. In this tech-note, we will see how IPv6 packets with Extension Header can also be filtered using UDF. 

## What is IPv6 Extension Headers
