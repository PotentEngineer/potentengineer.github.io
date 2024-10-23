---
title: Automating Wireshark in Windows
description: For advanced scenarios where you need to automate running Wireshark at scale
date: 2024-10-22T00:16:01.062Z
preview: /images/PreviewAutomatingWireshark.png
tags:
    - configmgr
    - powershell
    - windows
    - wireshark
categories: []
draft: true
---

# Automating Wireshark in Windows

This post is the second of three for automating common debugging tools on Windows endpoints. 

- [Automating Performance Monitor in Windows](https://potentengineer.com/2024/10/12/automating-performance-monitor-in-windows.html)
- [Automating Wireshark in Windows](https://potentengineer.com/2024/10/12/automating-wireshark-in-windows.html)
- Automating Process Monitor in Windows

From the prior post:
*Earlier this year I came across a scenario of an application dropping connections. This was occuring across many hundreds of users and sporadically. Typically, I would attempt to recreate the issue so I could debug, but that was not possible here. I needed a way to be ready for the drop to occur and have all debugging tools setup proactively across a large number of users.*

*We use ConfigMgr to run scripts on workstations from a central location and it worked well in this scenario.*

## Initial setup process
This is where running this with ConfigMgr becomes convenient. Just target a collection with the software push and the script.

1. Package up Wireshark in ConfigMgr to be pushed silently
2. Add a Run Script to ConfigMgr to be ran on devices, Start-WiresharkCapture
3. Add the column for Script GUID to the ConfigMgr console and copy it out
   1. ![](/assets/images/ConfigMgrScriptGUID.png)
4. Create collection of target devices
5. Trigger the Run script each morning at a specified time on target workstations to create and start the capture using the [Invoke-CMScript](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/invoke-cmscript?view=sccm-ps) cmdlet. I used Azure Automation to trigger this daily and re-run hourly to ensure the Wireshark was always running. You can use any automation solution you prefer
   1. `Invoke-CMScript -CollectionId <CollectionId> -ScriptGuid <scriptguid>`

## The script
This automation was not difficult to setup. The [programmatic capabilities](https://www.wireshark.org/docs/wsug_html_chunked/ChCustCommandLine.html) of Wireshark made it fairly easy. The biggest challenge was cleanly ending the Wireshark processes and restarting, then ensuring only one instance was running at a time so that a capture was taken at all times.

Again, special thanks to my co-workers Darren Chinnon and Raul Colunga for the help on this one.

#### Script anatomy
1. Check for Wireshark.exe in default path
2. Check for c:\temp\capture path and create if necessary
3. Check if Wireshark.exe is running already
4. If Wireshark is not running, start it
   1. Gather the network interface alias of the interface running DHCP and connected
   2. Concatenate the Wireshark process with the network interface alias
   3. Start Wireshark with the identified alias
4. If Wireshark is running, check process count >=2
   1. If multiple processes running, kill processes
   2. Start Wireshark following the logic above in step #4

#### Partial Preview
```PowerShell
if (Test-Path 'C:\Program Files\Wireshark\Wireshark.exe') { # Check if Wireshark is installed
    if (!(Test-Path c:\temp\capture)) { # Check if c:\temp\capture exists
        New-Item -Path c:\temp -Name capture -ItemType Directory -Force # Create capture directory
    }
    
    $WSProcess = (Get-Process -Name Wireshark -ErrorAction SilentlyContinue)

    if ($WSProcess -eq $null) {
        # Gather the interface number
        $IntAlias = Get-NetIPInterface -AddressFamily IPv4 -ConnectionState Connected -Dhcp Enabled | Select-Object -ExpandProperty InterfaceAlias
        $IntListraw = & "C:\Program Files\Wireshark\Wireshark.exe" -D | Out-String
        $IntList = ($IntListraw.Split("`n"))
        $wsIntName = $IntList | Select-String -SimpleMatch ('(' + $IntAlias + ')')
        $wsIntNumber = $wsIntName.ToString()[0]

        Write-Output 'Wireshark not running, starting Wireshark.'

        # Start the capture
        & "C:\Program Files\Wireshark\Wireshark.exe" -i $wsIntNumber -b filesize:100000 -k -w "C:\temp\capture\$($env:username)-$($env:computername).pcapng"
    } elseif ($WSProcess.count -ge 2) {
        Write-Warning 'Multiple Wireshark processes running, killing, and starting Wireshark!'
        Stop-Process -Name dumpcap -Force
        Start-Sleep -Seconds 5
        Stop-Process -Name wireshark -Force
        
        # Gather the interface number
        $IntAlias = Get-NetIPInterface -AddressFamily IPv4 -ConnectionState Connected -Dhcp Enabled | Select-Object -ExpandProperty InterfaceAlias
        $IntListraw = & "C:\Program Files\Wireshark\Wireshark.exe" -D | Out-String
        $IntList = ($IntListraw.Split("`n"))
        $wsIntName = $IntList | Select-String -SimpleMatch ('(' + $IntAlias + ')')
        $wsIntNumber = $wsIntName.ToString()[0]

        # Start the capture
        & "C:\Program Files\Wireshark\Wireshark.exe" -i $wsIntNumber -b filesize:100000 -k -w "C:\temp\capture\$($env:username)-$($env:computername).pcapng"
    } else {
        Write-Warning 'Wireshark is already running, not starting a new instance!'
    }
}

Github link: [Start-WiresharkCapture.ps1](https://github.com/PotentEngineer/LabScripts/blob/master/Applications/Start-WireSharkCapture.ps1)
```
## Output
Starting Wireshark for the first time
![](/assets/images/WiresharkStart1.png)

Starting Wireshark when it is already running or running multiple instances
![](/assets/images/WiresharkStart2.png)

## Closing
This script was a bit more of a challenge than the Perfmon script, but allowed Wireshark to capture all network traffic at all times. This lead to the resolution of the issue we were experiencing, so it paid off. 