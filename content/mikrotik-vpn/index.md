+++
title = "VPN IPSec IKEv2 with Mikrotik router"
date = 2025-01-07

[taxonomies]
categories = ["Mikrotik"]
tags = ["code","mikrotik","network","vpn"]
+++

Mikrotik routers are powerful devices with a lot of features for various networking needs. In this post I describe how to setup IPSec IKEv2 VPN with RouterOS, that could be used to remotely access organizational or home network. 
<!-- more -->
Building your own home VPN might be very useful - for example if you have some PC connected to your router, you can access it anytime. If your ISP provides you dynamic IP, you can use [Dynamic DNS](@/dyndns-ovh-mikrotik/index.md) to use DNS name to access your home network.

First, we have to generate certificates.

todo