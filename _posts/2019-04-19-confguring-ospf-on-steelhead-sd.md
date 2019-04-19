---
layout: post
title:  "Configuring OSPF on SteelHead SD"
---

# Configuring OSPF on SteelHead SD

Hi there.  I'm Brandon Carroll, one of the Technical Evangelists at Riverbed.  In this post I'm going to show you how to configure OSPF on a SteelHead SD.  You're not going to configure OSPF directly on the device, because a SteelHead SD is an SD-WAN Gateway.  So, in true form for a "Software Defined" solution, we are going to do everything on the controller.  Our controller is cloud-based more than 90% of the time.  Rather than refer to it as a controller, I'm going to refer to it as SteelConnect Manager, or SCM for short.

## Logging into SCM
