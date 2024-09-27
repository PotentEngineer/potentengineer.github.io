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

Even if you haven't been paying attention to recent development or lack thereof Microsoft Configuration Manager or to any of the threads on Twitter/X, Reddit, or any other major social media platforms, you still probably know the writing is on the wall for ConfigMgr. Nearly all focus in Contosoland has been devoted to Intune. 

This post outlines my personal running list of gaps that Intune doesn't quite cover for the seasoned ConfigMgr administrator. 

My friend and banter extraordinaire, Bryan Dam, [posted recently](https://x.com/bdam555/status/1825882130515128778) a quote that makes sense in describing this list. 

*#ConfigMgr gave you 250% of what you need. #Intune gives you 90%, we'll get it to 100% ... eventually.*

Much of this list may be in the last 150%, but that doesn't change a lot of organizations' business requirements. Make your own determination how critical these capabilities are for your organization. 

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
For the majority of your software installations, Intune should cover your needs. But the following requirements may pose an issue. 
* **Sequencing complex installs together** - Need to deploy multiple installs in a certain order or tie in installs, scripts, and restarts at once? Complex installs like Citrix VDA or sometimes Windows Feature Upgrades are easily handled with ConfigMgr task sequences. No equivalent in Intune without complex PowerShell scripting. 
* **30GB maximum package size** - This may pose an issue for things like Visual Studio, AutoCAD, SAS, and other very large applications. Your best option is to compress the install files into a .zip or .wim file and extract them at install time. 
* **IME runs as 32-bit not 64-bit** - For the majority of software installs, this is no issue. Windows Installer is intelligent enough to install 64-bit to 64-bit, even when called from a 32-bit process. However, if you use PowerShell to wrap your installs, such as the ever popular PowerShell Application Deployment Toolkit, you have to take extra steps to trigger the process as 64-bit. Not ideal, especially if you wrap all things in PSADT. 

### Shared device scenarios
* **Non-persistent VDI scenarios** - Want to go full modern Entra ID only + Intune only? [Not supported](https://learn.microsoft.com/en-us/entra/identity/devices/howto-device-identity-virtual-desktop-infrastructure#supported-scenarios), hard stop. To be fair, most orgs are probably not using ConfigMgr on non-persistent scenarios, but they are using GPO. So going Entra ID only is a killer here. Not exactly an Intune gap, but modern endpoint as a whole.
* **No maintenance windows** - There are lots of uses for admin-controlled hard coded timeframes when maintenance can occur. Known as Maintenance Windows in ConfigMgr, they are very valuable for highly critical devices, shared devices, and devices running on shared infrastructure like VDI. Based on recent conversations this does seem to be top of mind for Microsoft.

### Real time capabilities
* **8 hour policy check-in (ConfigRefresh is 90 minutes)** - ConfigMgr, formerly known as "slow moving software" is arguably much faster than Intune. Just check Reddit for comments about Intune being slow. Simpler? Maybe, but the admin can't exactly control when things will happen. Recent [improvements to ConfigRefresh](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/intro-to-config-refresh-a-refreshingly-new-mdm-feature/ba-p/4176921) have helped here, but it still can't beat a 60 minute policy refresh in ConfigMgr for all actions on the endpoint. What does our org do? We have a Run Script in ConfigMgr to trigger a MDM sync using the scheduled task. The forced sync from the Intune portal isn't reliable enough and cannot be performed in bulk. 
* **Expire/disable deployments** - It's 11pm at night, you just deployed something to thousands of endpoints. Your helpdesk's call volume starts to rise. Big red button time. In ConfigMgr, you never want to delete the deployment, you lose all history and record of potentially impacted devices. You instead expire or disable the deployment. In Intune, no such option. you remove the assignment and find another way to identify impacted endpoints. Can you parse the IME log, event log, Sysmon, MDE, or other means to see what occured on the endpoints locally? Sure, but that is way more complex and time consuming when leadership has you on an outage bridge asking for impacts now. 

### Maximum of 200 Remediation scripts
If you are a large organization, you may hit the 200 maximum [remediations limit](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations#script-requirements). Our organization has well over 200 ConfigMgr baselines, so we are keeping this workload in ConfigMgr for now. 

### Targeting
We all know that for modern endpoint management you generally want to target users, not devices. There is a lot of value targeting users, especially as Intune is designed to work better this way. However, you  lose out on certain deployment abilities that ConfigMgr delivers beautifully with collections today. Good news though, coming soon to Intune is [device inventory](https://techcommunity.microsoft.com/t5/microsoft-intune-blog/device-hardware-inventory-is-coming-soon-to-microsoft-intune/ba-p/4238042)! This seems to be the first step in opening up more targeting capabilities besides Entra ID groups and virtual groups + filters. 
* **Targeting based off installed software** - This is our most commonly used scenario. Nearly every software deployment we do follows this template. Collection of target devices excluding devices with X software installed. Build pilot groups off that collection. When your collection hits 0 you are done. It is a combination of targeting + inventory to maximize success of your software deployments. Our organization averages over 3000 software deployments a year, and every little software install success matters. 
* **Targeting based off installed software versions** - Same logic as above, but mainly for upgrades from version X to version Y.
* **Targeting based off software usage/metering** - Very valuable for software audits, license reclamation, and other software lifecycle scenarios. 
* **Targeting based off registry keys** - How many software applications do you have that store relevant info in the registry? Zscaler Private Access, Digital Guardian, lots of others. Inventory the registry keys, build a collection to target. 
* **Targeting based off WMI properties** - We have almost 100 custom inventory items we store in WMI to solve all our targeting wildest dreams. Drivers, some user profile specific registry key, custom branding, the list goes on. 
* **Targeting based off management properties (domain, co-managed workload, etc)** - This is mostly used for limiting deployments to relevant endpoints. Only want to target your Entra ID joined devices? Only hybrid joined? Only devices in a certain domain? All very valuable scenarios to ensure you limit exposure to devices that should only be targeted. 
* **Targeting null data such as software not installed** - This is valuable for cleaning up your environment. Missing a required configuration or devices that should have X software installed? 
* **Targeting based off policies (compliant, non-compliant, success, error, etc)** - If a software install fails, it will reattempt. What if you need to target a script to a device that failed to apply a config policy successfully? What if you need to apply a script to a non-compliant device?
* **Targeting based off user state (user logged on, primary user set, etc)** - One of the easiest ways to ensure you are not impacting an end user is if they are not logged into their device. This is great for overnight implementations and critical cleanup tasks. Being able to see the primary user of a device like with user device affinity in ConfigMgr is very valuable here. Great info for cross checking asset management systems and targeting users with multiple devices or accounts that login to multiple users devices. 

## Closing
This is a lengthy list, and I have been keeping this list since we co-managed all our devices at the start of the pandemic in 2020. The good news is even just a year ago, this last had 5 more items. Many of these items are dropping off with every monthly Intune release and eventually Microsoft will get that last 10%. I personally expect co-management will still be necessary for the next 5 years. We shall see. 