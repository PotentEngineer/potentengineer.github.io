---
title: Intune missing capabilities for the ConfigMgr administrator
description: This is my running list of capabilities that ConfigMgr delivers but Intune does not.
date: 2024-09-24T01:00:53.651Z
preview: ""
tags: []
categories:
    - intune
    - configmgr
draft: true
---
# Intune missing capabilities for the ConfigMgr administrator

Even if you haven't been paying attention to recent development of Microsoft Configuration Manager or to any of the threads on Twitter/X, Reddit, or any other major social media platforms, you still probably know the writing is on the wall for ConfigMgr. Nearly all focus in Contosoland has been devoted to Intune. 

This post outlines my personal running list of gaps that Intune doesn't quite cover for the seasoned ConfigMgr administrator. 

My friend and banter extroidenaire, Bryan Dam, [posted recently](https://x.com/bdam555/status/1825882130515128778) a quote that makes sense in describing this list. 

*#ConfigMgr gave you 250% of what you need. #Intune gives you 90%, we'll get it to 100% ... eventually.*

Much of this list may be in the last 150%, but that doesn't change your business requirements. Make your own determination how critical these capabilities are for your organization. 

[Kim Oppalfens](https://x.com/thewmiguy) has the [best write up]( https://www.oscc.be/sccm/configmgr/Making-the-case-for-cloud-attach-and-co-management/) I have seen to date of these gaps at a higher capability level. 

- Realtime scripts
- Realtime application installation
- CMPivot
- Task Sequences with Autopilot
- Custom inventory
- Flexible targeting
- Configuration baselines
- Software metering
- Customizable reporting
- Package distribution

My list includes a few more technical gaps ranging from critical to minor technical details. 

### Software installation
Complex installs like ConfigMgr task sequences 
Software installation - specific/consecutive order 
Software installation - 30GB maximum size
IME runs as 32-bit not 64-bit 

### Non-productivity worker scenarios
Non-persistent scenarios
Patching capabilities, only WufB (no maintenance windows) 

### Real time capabilities
8 hour policy check-in (ConfigRefresh is 90 minutes) 
Expire/disable deployments

### Platform scripts 200 max

### Targeting
Targeting based off installed software 
Targeti ng based off installed software versions 
Targeting based off registry keys 
Targeting based off WMI properties 
Targeting based off management properties (domain, co-managed workload, etc) 
Targeting null data such as software not installed 
Targeting based off policies (compliant, non-compliant, success, error, etc) 
Targeting based off user state (user logged on, primary user set, etc) 