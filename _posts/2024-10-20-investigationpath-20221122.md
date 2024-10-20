---
layout: post
title: "InvestigationPath 20221122"
date: 2024-10-20 18:32:00 -0000
categories: investigationpath
---

- [InvestigationPath 20221122](#investigationpath-20221122)
  - [My Response](#my-response)
    - [Data Access](#data-access)
    - [Data Removal](#data-removal)
  - [Other Responses](#other-responses)
  - [Resources](#resources)

# InvestigationPath 20221122

The next scenario is a classic HR one ([link](https://x.com/chrissanders88/status/1595070232426467328)):

<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">Investigation Scenario 🔎<br><br>HR suspects that a former employee may have taken sensitive data when they quit.<br><br>What do you look for to investigate this event?<br><br>Assume you have access to any evidence source you want, but no commercial DLP tools.<a href="https://twitter.com/hashtag/InvestigationPath?src=hash&amp;ref_src=twsrc%5Etfw">#InvestigationPath</a><a href="https://twitter.com/hashtag/DFIR?src=hash&amp;ref_src=twsrc%5Etfw">#DFIR</a> <a href="https://twitter.com/hashtag/SOCAnalyst?src=hash&amp;ref_src=twsrc%5Etfw">#SOCAnalyst</a></p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1595070232426467328?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So again, here we're attempting to investigate a potential data theft scenario. We'll operate under the assumption that their endpoint is still available and hasn't been either stolen or wiped. We'll also assume the user had a Windows workstation.

## My Response

**Limitation:** No commercial DLP available.

As for what to look at, I think of a potential data theft scenario consisting of two parts:

1. Access to sensitive data
2. Removal of sensitive data

Before we begin it's probably best to set expectations and remind HR of our capabilities. It's unlikely that we'll be able to uncover any "smoking gun" or definitive evidence one way or another. Sensitive data theft can be hard to prove, for example if the user took a picture of their screen with their phone then we're not going to have evidence of that.

If HR suspects data theft then they should be able to provide some additional context, such as why they suspect was stolen and / or why. It's important to ask for this information because it can help focus an otherwise broad investigation.

### Data Access

For data to be stolen it probably has to be accessed. This data can probably live in a number of different locations and be accessed in a number of different ways. It's good to start by listing what sensitive information is available, where it is stored, and how it can be accessed. This provides us with guidance on what to look for, where, and how.

Data is probably stored either on your network (meaning on a workstation or server), or in the cloud (meaning a cloud service like AWS or Azure or a SaaS product).

Data can be accessed either through an application on the workstation, a web interface like Microsoft 365 products, or remotely via an unmonitored endpoint. The way that it is accessed is probably (but not necessarily) linked to the way / location that it is stored.

If the data is accessed via an application on the endpoint then we can find evidence in the system's "shellbags" which contain evidence about what files and folders were accessed on the system and when.

Access to data that is accessed online (such as in an S3 bucket or in Microsoft 365) may be logged by the hosting application. For example, you could look at the user's activity in Microsoft's Unified Audit Log to see what files they accessed and when. Depending on the services that you have in use, investigate their logging capabilities to determine whether or not you can investigate data access. Document your findings so you know next time.

### Data Removal

While EDR generally doesn't record file READ events, it does capture WRITE events (with limitations depending on what EDR you're using). We can check EDR for file write events that either contain interesting names, extensions, or to interesting locations (like non-`C:\` locations). We can also check it for processes or network traffic to sites that could be used to exfiltrate data.

The registry will also contain evidence of USB devices that were plugged in. If your EDR doesn't log USB devices or the relevant registry modifications, then consider pulling the evidence directly from the device (in a forensically sound manner) for investigation. Of course, if the user took a picture of their screen, then you're not going to have any logging available to demonstrate that.

## Other Responses

There are so many different ways that determined users try to steal data, the responses to this tweet had a lot of good thoughts.

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">1. Identify what the data is, where it is stored, how it can be accessed (file share, internal portal, etc.)<br>2. Check corresponding events (file access, browsing, shell bags, etc.)</p>&mdash; Mehmet Ergene (@Cyb3rMonk) <a href="https://twitter.com/Cyb3rMonk/status/1595096312344723460?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">- Browser history for things like Dropbox, mega NZ<br>- USB plug-in dates from C:\windows\inf\setupapi.dev.log<br>- SRUM forensics and spike in outbound data<br>- VPN sessions outside of work hours, and contextualise the above data’s timestamps</p>&mdash; Dray Agha (@Purp1eW0lf) <a href="https://twitter.com/Purp1eW0lf/status/1595072568662310912?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">I have had this exact scenario in the past, look at that user activity, compared it to previous weekends. O365 sharepoint logs was where we found everything. Spent the weekend downloading company data.</p>&mdash; baked beans (@beans1990) <a href="https://twitter.com/beans1990/status/1595141873446051840?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Lots of great answers here. Didn&#39;t see a lot of email logs. Many insiders email themselves documents.</p>&mdash; Chris Prewitt (@cp_cto) <a href="https://twitter.com/cp_cto/status/1595158740197863424?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">First find out what’s allowed or blocked, ie if USB are blocked or webmail, etc.</p>&mdash; -WK (@warren_kruse) <a href="https://twitter.com/warren_kruse/status/1595091663008980994?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">First place I&#39;m looking is email...web content filtering logs...usb connections in the registry...and we had commercial DLP in place because I put it in (most of the time it was us telling HR someone took data, usually right before they quit).</p>&mdash; Dan Kennedy 🚫 (@danielkennedy74) <a href="https://twitter.com/danielkennedy74/status/1595198174171467777?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Before I dive into any analysis there are a few key questions I would like to answer which could help determine scope:<br><br>What evidence does HR have for their suspicions?<br><br>What role was this user in?<br><br>What access did they have?<br><br>If they quit for a new role, which company is it?</p>&mdash; L0Psec (@L0Psec) <a href="https://twitter.com/L0Psec/status/1595415386593415172?ref_src=twsrc%5Etfw">November 23, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">I&#39;d try and figure out what data they had access to, combined with who they&#39;d been in contact with to try and establish a motivation/end goal for taking sensitive data. Maybe they met a competitor and saw an opportunity with them. Looking at their social media can help</p>&mdash; KF_Lawless (@KF_Lawless) <a href="https://twitter.com/KF_Lawless/status/1595176155807637504?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">I&#39;d try and figure out what data they had access to, combined with who they&#39;d been in contact with to try and establish a motivation/end goal for taking sensitive data. Maybe they met a competitor and saw an opportunity with them. Looking at their social media can help</p>&mdash; KF_Lawless (@KF_Lawless) <a href="https://twitter.com/KF_Lawless/status/1595176155807637504?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">1. Clipboard data to determine what May have been copied from sensitive content<br>2. Unsaved Office documents that could contain sensitive information<br>3. User Assist key to see what the user accessed leading up to date of departure/termination</p>&mdash; CyberReverend (@ComandanteBowie) <a href="https://twitter.com/ComandanteBowie/status/1595110190210199553?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">4. Removable media connections with staged data that could have been burned<br>5. Email/PST info to see if sensitive was saved and sent out via email<br>6. Browser history to look for any file uploads to an external email/website</p>&mdash; CyberReverend (@ComandanteBowie) <a href="https://twitter.com/ComandanteBowie/status/1595110584152125441?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">- Get more info from HR on user&#39;s role and expected access (i.e. Sales guy accessing Engineering share is weird)<br>- Browser logs to find traffic to sharing sites <br>- Mail logs to see if user mailed anything to self<br>- Jump List + LNK for USB forensics<br>- Zeek to correlate bytes out</p>&mdash; Droogy (@0xDroogy) <a href="https://twitter.com/0xDroogy/status/1595121946122821632?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">- Get more info from HR on user&#39;s role and expected access (i.e. Sales guy accessing Engineering share is weird)<br>- Browser logs to find traffic to sharing sites <br>- Mail logs to see if user mailed anything to self<br>- Jump List + LNK for USB forensics<br>- Zeek to correlate bytes out</p>&mdash; Droogy (@0xDroogy) <a href="https://twitter.com/0xDroogy/status/1595121946122821632?ref_src=twsrc%5Etfw">November 22, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Resources

- [Shellbags Analysis by Chris Eastwood](https://medium.com/ce-digital-forensics/shellbag-analysis-18c9b2e87ac7)
- [Making Sense of ShellBags by Ross Andrews](https://medium.com/@andrewss112/making-sense-of-shellbags-8a8e945d8f2d)
- [Eric Zimmerman's ShellBags Explorer too](https://ericzimmerman.github.io/#!index.md:~:text=exporting%20shellbag%20data-,ShellBags%20Explorer,-%2D%20%7C%202.0.0.0)
- [Identifying Usage of a USB Storage Device: A Forensic Guide Using Windows Registry by mumblesec](https://medium.com/@mumblesec/identifying-usage-of-a-usb-storage-device-a-forensic-guide-using-windows-registry-a4c1db90b1ce)
- [USB Device Forensics by Janet Smith](https://www.asdfed.com/USB-Device-Forensics)
