---
title: Automating Performance Monitor in Windows
description: For advanced scenarios where you need to automate running Perfmon at scale
date: 2024-10-12T19:16:49.784Z
preview: /assets/images/PreviewAutomatingPerfmon.png
tags:
   - perfmon
   - powershell
   - windows
   - configmgr
categories: []
---

# Automating Performance Monitor in Windows

This post is the first of three for automating common debugging tools on Windows endpoints. 

- [Automating Performance Monitor in Windows](https://potentengineer.com/2024/10/12/automating-performance-monitor-in-windows.html)
- Automating Wireshark in Windows
- Automating Process Monitor in Windows

Earlier this year I came across a scenario of an application dropping connections. This was occuring across many hundreds of users and sporadically. Typically, I would attempt to recreate the issue so I could debug, but that was not possible here. I needed a way to be ready for the drop to occur and have all debugging tools setup proactively across a large number of users.

We use ConfigMgr to run scripts on workstations from a central location and it worked well in this scenario. 

## Initial setup process
1. In Performance Monitor (perfmon.msc), create the necessary [Data Collector Set](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/create-data-collector-performance-counters), henceforth abbreviated DCS
   1. Be sure to specify the directory you want your output in
2. Export the DCS you created. Right-click and Save template to .xml.
3. Download Start-PerfmonCapture.ps1 linked below
4. Copy the contents of the .xml into below script so no artifacts are necessary outside of the script. It is self-contained.
5. Add a Run Script to ConfigMgr to be ran on devices, Start-PerfmonCapture
6. Add the column for Script GUID to the ConfigMgr console and copy it out
   1. ![](/assets/images/ConfigMgrScriptGUID.png)
7. Create collection of target devices
8. Trigger the Run script each morning at a specified time on target workstations to create and start the DCS using the [Invoke-CMScript](https://learn.microsoft.com/en-us/powershell/module/configurationmanager/invoke-cmscript?view=sccm-ps) cmdlet. I used Azure Automation to trigger this daily and re-run hourly to ensure the DCS was always running. You can use any automation solution you prefer
   1. `Invoke-CMScript -CollectionId <CollectionId> -ScriptGuid <scriptguid>`

## The script
This script is fairly basic, mostly just took figuring out how to interact with a Data Collector Set using logman.exe and make sure it didn't clobber the existing DCS that is running.

Special thanks to [Aussie Rob SQL](https://www.aussierobsql.com/), [Jonathan Medd](https://www.jonathanmedd.net/), and [Rabi Achrafi](https://rabiachrafi.wordpress.com/) for the example scripts I found online. References are in the script help text. Also thanks to my co-workers Darren Chinnon and Raul Colunga who helped put this together.

#### Script anatomy
1. Populate the XML for the Data Collector Set
2. Specify name of DCS
3. Query if DCS already exists with specified name
5. If DCS found, check if running
   1. If running, exit
   2. If not running, start it
6. If DCS not found, create it and start it

Edit the lines below to personalize as needed

- Line 27 through 210 - Your custom XML
- Line 212 - DCS name

#### Partial Preview
```powershell
$DCSName = 'PerfMonExample'
$DCSCheck = & logman query $DCSName # Query if DCS already exists

if ($DCSCheck[1] -like "*$($DCSName)") {
    Write-Output 'DCS found!'
	if ($DCSCheck[2] -like '*Running') {
		Write-Output 'Trace running, exiting...'
	} else {
		Write-Output 'Trace not running, starting...'
		& logman start $DCSName
	}
} else {
    Write-Output 'DCS not found, creating...'
    
    # Create the Data Collector Set
    $DCS = New-Object -COM Pla.DataCollectorSet
    $DCS.DisplayName = $DCSName
    $DCS.SetXml($DCSTemplate)
    $DCS.Commit("$DCSName" , $null , 0x0003)

    # Start the data collection
    Write-Output 'Starting the DCS!'
    $DCS.start($false)
}
```

Github Link: [Start-PerfmonCapture.ps1](https://github.com/PotentEngineer/LabScripts/blob/master/Applications/Start-PerfmonCapture.ps1)

## Output
The script gives a little output for validation, not much though. This is mostly for validation during testing.

Starting the script, there may be additional output from logman.exe for the initial run
![](/assets/images/PerfmonStart1.png)

Starting the script when DCS is already running
![](/assets/images/PerfmonStart2.png)

Starting the script when DCS is not running
![](/assets/images/PerfmonStart3.png)

## Closing
Overall, this process worked well and met the need. It wasn't the first time I had to use Perfmon and it won't be the last. Up next, Wireshark.