---
title: Managing endpoint policies for the enterprise
description: This post provides real-world guidance on managing Windows endpoint policies in a large enterprise. These concepts may be applied other endpoints as well. 
date: 2025-07-02T02:23:51.171Z
preview: /images/ClipboardGear.jpg
tags:
    - policy
    - pilot
categories: []
draft: true
---
# Introduction
This one goes out to Tay, also known as [SwiftOnSecurity](https://bsky.app/profile/swiftonsecurity.com). We had a discussion in late 2024 around policy management in the enterprise, so here we are. 

I have been deploying OS, app, device, and *whatever else comes along* policies in mid to large enterprises for most of my career. They say the best way to learn is through failure and I have had my fair share of outage causing failures with many a lesson along the way. 

My first big lesson was always testing scripts before deploying them. This is a simple lesson that is known by most at this point, but valid nonetheless. Mix equal parts confidence, excitement, lack of experience, and a missing parameter and boom, you wipe out ~300 nurse managers user profiles trying to clean up some disk space. This was one of my first big deployments.

My next big lesson was not to modify active production deployments. I was helping a co-worker troubleshoot a ConfigMgr collection that didn't have any members. Easy, right? Until you realize that collection you just fixed had a required task sequence deployment with multiple restarts in it. What did the collection members jump to? Only about 10,000 workstations and servers. After the unstoppable 6 hour outage, I was terrified explaining that one to our VP. This outage led to the split of our single ConfigMgr environment into five separate workstation and server environments later that year, which I am thankful for today. 

These lessons, along with many others, have shaped the robust set of practices outlined below. Our teams apply these practices across multiple endpoint management tools to minimize risk, avoid impacting user experiences, and prevent outages. Whether we’re using Group Policy, Intune, ConfigMgr, JAMF, or other tools, these guidelines help keep everything running smoothly.

# Environment setup
### Lower environments
- Test, Dev, Staging, Int, QA - Whatever you want to call it, setup a lower environment. It is up to you whether you need one or many. 
  - Your lower environment should mirror Production as best as possible. Are you really testing if your environment doesn't match?

*Everyone has a test environment, some are just in Production.*

### Deployment targets
- Setup pilots, rings, exclusions, and segregation of critical endpoints. Examples below.
  - You should "slow-roll" all deployments to reduce risk. This ensures impact is minimized for any unknowns. Start small and ramp up deployments to more endpoints. Even if we are highly confident there is no impact, the risk of a single night's deployment is generally too high.
  - If you have endpoints in lower environments, target those first. You should also prioritize any dedicated testing endpoints like test servers. 
    - Virtual machines are very handy here. One of our common test targets are reclaimed/unused VMs that are set to be decomissioned. Well used machines with no user impact? Yes please!
  - Setup testers for your most critical applications. 
    - In our environment, we have a group of non-IT volunteers that test changes and new technologies early to ensure no impacts. These are production users on production workstations. I cannot express enough the value we get from this. It is the closest we can get to ensuring there are no impacts. 
  - You should move your critical endpoints to the end of your deployment schedule. Even if you ramp down like 500, 1000, 5000, 250.
  - Develop maintenance windows where necessary. Deploy to workstations during non-business hours. Deploy to servers during lowest usage hours. Work with the owners of your endpoints to determine the best time to deploy. 
    - In our server environment, server owners can actually choose their maintenance windows directly with the patching team. They absolutely love having this freedom. 
  - Reuse pilot groups where possible. Generally, you will always have deployments that need to target all devices. This is a great way to have pre-populated pilot groups that you can reuse for these deployments. Your first pilot can include those volunteers and critical app testers so that you ensure any impacts are identified early on. 
  - Separate endpoints into logical buckets to eliminate points of failure. A simple example is splitting workstations and servers out. You could separate by configuration, network connection, department. Use what works for your environment. 

### Least privilege and separation of duties 
- Follow the principles of least privilege. Accidents happen, we are all human. Restrict those who should not have permission to deploy to all devices, all users, all servers, etc. 
  - For our critical Production deployment systems, we have a two step process as part of onboarding a new engineer.
      1. Access to lower environment first for two weeks for training/shadowing.
      2. A test/quiz that must have 80% success before granting Prod access with a meeting to review results. 
  - Newly onboarded engineers should not handle critial deployments for a period of time until they are familiar and comfortable with that level of potential impact. 
- Use seperation of duties where possible. Only experienced engineers should handle the most critical deployments. A common mistake is a new engineer joining a team and immediately getting far-reaching privileges and breaking something. This isn't the engineers fault, but the process that allowed the engineer to break something.   
    - The person handling implementation should not be the one performing validation. Heva the requestor perform validation or at least a separate member of the deployment team. 

# Standards and change control
### Documentation
In order for these practices to be effective, they need to be documented so that the value is well known and they can be enforced. Awareness is your friend, especially in urgent and emergency circumstances. If your requestors and leaders are not aware of these processes, they generally won't be happy when they want something to happen now. 
- Formalize all of your processes. You can make an official Operational Standard using your enterprise wide Standards process if you have one. If you do not need that level of formality at least put the process in writing in your document or knowledge management system. 
- Develop metrics for barriers and safety nets. Include these metrics in your standards and documentation for more awareness. Some examples below. 
  - What constitutes a high risk change vs. medium or low?
  - What are the maximum number of devices you can deploy to in a single night?
  - How many test devices must be successful to move to production?

### Change control
Another method to enforce these standards is through a change control process. [ITIL](https://wiki.en.it-processmaps.com/index.php/Change_Management) is a popular choice. I don't think anyone likes change control, but it is a necessary evil. In many organizations, change control is an enterprise requirement, so you don't always have to be the bad guy.
- One benefit is that once your process is mature enough, you can start to simplify. This is where **Standard** changes come in. They usually have less paperwork, a template, or automatic approvals. 
  - In our environment we use standard changes for our Windows in-place upgrades, monthly patching, driver deployments, policy changes that do not impact user experience (as opposed to those that do), and software deployments to less than ~5% of our workstation fleet (~5000 devices). All of these were possible because we had already completed hundreds of each of these changes using the **Normal** change process. The process was proven and rock solid so we were allowed leniency.

# Preperation, Implementation, and validation
### Automation
One of the most disappointing feelings is when you miss a simple checkbox, step, make a syntax error. The simple things. Where it makes sense, you should automate these steps. Even just a simple script to make sure all boxes were checked can be a lifesaver one day. It only takes one time for a script to catch something to prove it's worth. 

Here are a few examples of easily missed implementation steps where automation can help.
- ConfigMgr - Editing task sequences, especially large ones. Run Command Line steps, Run PowerShell steps, adding conditions. Very easy to typo here.
- Group Policy - New linked policies automatically target authenticated users. Yep, that is everyone. 
- Intune - Targeting the All Devices virtual group, but forgetting to add your filter. Yep, that is also everyone. 

For mature environments, don't touch prod, use Continuous Integration/Continuous Deployment (CI/CD) pipelines. Human's make the changes in the lower environment, scripts turn them to Prod. There are some great tools to help with this like [IntuneCD](https://almenscorner.github.io/IntuneCD/) (Tobias Almén) and [M365DSC](https://microsoft365dsc.com/) (Microsoft). 
### Implementation preperation 
Data and visibility are key to helping eliminate unknowns and reduce risk. If you need to change software or configurations, first inventory your environment to see what it looks like currently. Do not rely on assumptions or accept unknowns where possible.
##### Tooling
There are many tools that can provide visibility into your environment and help decision making. Use them. Many environments have the tools in place but they are highly underutilized. 
- Endpoint Management tools - ConfigMgr, Intune, JAMF, Workspace One, and others
  - Inventory of software, files, settings, and more
  - Software metering and usage to see what software is in use
  - Real-time capabilities with CMPivot, Run scripts, Platform scripts, etc. 
- Digital Employee Experience (DEX) tools - Nexthink, 1E, Lakeside, etc. 
  - Inventory of software, files, settings, and more
  - Real-time capabilities with remote actions and more
- Endpoint Detection and Response tools - Defender, Crowdstrike, and the like
  - Inventory from a security perspective like software, patching, vulnerabilities
  - Real-time capabilities with running scripts
  - Advanced event logging
- Security Information and Event Management (SIEM) tools - Sentinal, Splunk, Sysmon, and more
  - Indirect inventory through logging and usage
  - Advanced event logging
- Observability platforms - Solarwinds, Dynatrace, Datadog, etc
  - Indirect inventory and usage through monitoring
- Scripting - PowerShell, Bash, Python, etc
  - They may not be able to get you the full picture, especially at scale, but some visibility is better than nothing. Get a subset of data and extrapolate from that. Just make sure not to break anything.

Sometimes data in one tool might be incomplete or even inconsistent, cross-reference tools if you have to. Here are a few examples.
- I need to see users who have used Chrome browser. I query software metering with ConfigMgr and cross reference it with Defender + Sysmon for process execution. 
- I need to see users who visited a website to build an accurate user list. I can use Nexthink for site visits, but I need page details. I can get that from Dynatrace. Using both paints the complete picture. 
- I need to see plug and play events for USB devices. Event log data in Splunk has some details, but Defender exposes more events. 

##### Determining the need for change
What is better than covering all your bases and having a flawless implementation? Not need to implement that change in the first place. Use the tools above to make the most educated decisions. You may not need to introduce as much risk or potential impact as you expected. I have had many direct experiences where data saved the day and prevented change. For well-planned changes this is helpful, but it also can save the day in emergency situations. Below are a couple examples. 
- Review software installations and usage. How many users have this installed? Of that, how many actually use it? This data alone can help determine the need for change. 
  - e.g.) We recently did an exercise to reduce the software load in our base image. We found a few of the applications were used by less than 5% of our users, even though business leaders claimed these applications were critical and had high usage. The data won out.
- Use tools at your disposal to query registry, WMI, files and folders, and many other things to determine the need for change. 
  - e.g.) A few years back we were looking to make a browser change. I used CMPivot to query the specific registry key and found over 2/3 of our environment already had it set because Microsoft had turned it on by default. Now we knew there was little risk to implement on the remaining 1/3. 
##### Preparing for implementation
When you are ready to move forward with implementing something new, save future you some trouble, and do as much preperation as possible before you implement. This means preparing the actual scripts, paperwork, and the actual change beforehand. If you are scheduled to implement at 7pm, do not sign-on at 7pm and start getting everything prepped. Do it the day before or the day of, so that implementation can be just that and nothing more. 
- Schedule deployments beforehand, where possible. Most endpoint management tools nowadays allow you to do this. 
- Prepare all paperwork beforehand, such as documentation, change controls, and communications to end users or stakeholders.
- Manage your time. If your sign-off at 5pm, do you have a reminder or calendar entry to alert you to sign-on at 7pm? This one is more important the more extreme the implementation time is. Some organizations require changes at 2am or on Saturdays, or even variable times based on lowest usage points. 
- Ensure you have all the support you need. Should a co-worker be there? Do you have a separate person handling validation? Should the vendor be on the call? Early on, when we used to do our ConfigMgr upgrades, we always had our Microsoft PFE/DSE/CSE on with us in case of issues. 
- If you are automating certain steps, please write that script up before implementation starts. You never know when that 5 minute task will turn into a 60 minute ball of frustration. 
### Validation
Validation is ensuring that your change was successful. Successful means that it did what you expected it to. You can successfully install software and also have unforeseen negative impacts. 
- Validation is always best done by a separate person. Ideally, the requestor for the change should handle validation. They are the ones that must be happy in the end. This will depend on whether they are another person in IT, have permissions to validate, and other environmental circumstances. 
- Validate not only that the deployment status was successful, but the actual change on the endpoint is reflected such as software being installed or registry being changed.
- Use the tools listed above to perform validation. Something as simple as querying for the appropriate registry key after making a GPO or Intune change goes a very long way. 
- Query all your targeted devices. If you are deploying to 500 devices, how comfortable are you that the 2 you manually checked reflect the other 498? Again, use the tools available to you. 
  - For all of our AD/EntraID deployment pilot groups for GPO/Intune changes, we create collections in ConfigMgr that sync the members so we can easily query them after each change using real-time tools like CMPivot. 
# Summary
This is a very lengthy post, but I did not want to separate it into multiple posts, so here is a summary of the major points below. 
- Always test scripts and avoid modifying active production deployments to prevent outages.
- Use lower environments that closely mirror production for safe testing.
- Deploy policies gradually using pilots, rings, and maintenance windows; prioritize critical endpoints last.
- Apply least privilege and separation of duties to minimize risk from human error.
- Document all processes and metrics to ensure awareness and enforce standards.
- Use change control (e.g., ITIL) to manage and simplify changes.
- Automate repetitive or error-prone steps to reduce mistakes.
- Inventory and analyze your environment before making changes; use multiple tools for complete visibility.
- Only implement changes when truly necessary, using data to guide decisions.
- Prepare all scripts, paperwork, and communications before implementation.
- Validate changes thoroughly, ideally by a separate person, and use tools to confirm success across all targeted devices.

# Conclusion
All the practices outlined above are lessons learned through trial and error. Some of them I may not have been the largest supporter of at first, but over time I came to see the benefit which ended up in me and my organization's favor. You may not need all of these, but consider what makes sense in your environment. 

What are some of the ways you have reduced risk in your environment?

p.s, a thank you to the many folks that have helped build these processes along the way. Jerry Boehnlein, Jim Parris, Mike Cook, Darren Chinnon, Scott Forshey, Jason Luckey, Matt Wright, Jason Mattingly, Nick Combs, Tim Robinson, Scott Hublar, Rex Ference, Sam Himburg, Paul Humphrey, Danny Hutchinson, Shaun Poland, and probably a few more than I cannot remember. 