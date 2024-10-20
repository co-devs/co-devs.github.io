---
layout: post
title: "InvestigationPath 20221108"
date: 2024-10-20 03:32:00 -0000
categories: investigationpath
---

- [InvestigationPath 20221108](#investigationpath-20221108)
  - [My Response](#my-response)
    - [Windows Event Logs](#windows-event-logs)
  - [Other Responses](#other-responses)
    - [Notes From Other Accounts](#notes-from-other-accounts)
    - [Notes From Chris](#notes-from-chris)
  - [Resources](#resources)

# InvestigationPath 20221108

If we take them chronologically, then the first scenario would be from 08 November 2022 ([link](https://x.com/chrissanders88/status/1589996232889360384)):

<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">Investigation Scenario 🔎<br><br>A workstation attempted authentication to every other Windows system on the local network.<br><br>What do you look for to start investigating this event?<br><br>Assume you have access to any evidence source you want, but no commercial EDR tools.<a href="https://twitter.com/hashtag/InvestigationPath?src=hash&amp;ref_src=twsrc%5Etfw">#InvestigationPath</a></p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1589996232889360384?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

So the goal here is to identify what to look for to kick off an investigation.

## My Response

**Limitation:** No commercial EDR available.

My first thought is how do we know this happened? This could help us to determine what information is already known and provide us with some useful context to begin the investigation. The scenario doesn't provide this answer, though, and I don't want to add too much on to it, we'll assume this information came from a coworker in IT and that they can provide more information if asked.

My next thought is that this could be caused by so many different things. Before going too deep down any rabbit holes, I want to try to determine if this activity is malicious or benign. Frequently, this can be answered by asking the question "does it make sense for this resource to be doing what it's doing?"

I'd want to start off by collecting the following information:

| Info                      | Reason                                                                                 |
| ------------------------- | -------------------------------------------------------------------------------------- |
| Hostname                  | Basic information                                                                      |
| System Role               | Might help explain the reason for the authentications or help define potential impact  |
| Source IP Address(es)     | Basic information                                                                      |
| Username                  | Basic information                                                                      |
| User Role                 | Might help explain the reason for the authentications or help define potential impact  |
| Type of Authentication    | Might help explain the reason for the authentications and provide a useful pivot point |
| Results of Authentication | Might help explain the reason for the authentications or help define potential impact  |
| Responsible Process       | Might help explain the reason for the authentications or help define potential impact  |

If my coworker doesn't know all of the above information, then I can likely pull it from Windows event logs. The scenario doesn't tell us if we have access to the system that is conducting the authentications or only the other systems. Since EDR isn't available, it's possible that this scenario is an example of a rogue endpoint of some kind (be it a legitimate host that wasn't properly enrolled in EDR or an adversary controlled system that obtained network access).

Finally, I wonder what the timing and frequency of this activity is. Is it happening during normal business hours or after? Is it spread out or all at once? And does it appear to be happening on some kind of recurring basis. This information could point me towards a legitimate tool / service on the workstation.

### Windows Event Logs

The scenario doesn't specify whether or not the authentications were successful. If they were not, then we'd want to look at Event ID 4625 in the Security log ([link](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625)). If successful, then we'd want to look at Event ID 4624 in the Security log ([link](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624)). In this case, we don't know, so we can filter on both event IDs. We should have some information for the source host, so we can also filter on either the hostname or IP address which should appear in the `Network Information` section of both event IDs. One of the most useful pieces of information will be the `logon type`, which is logged as an integer. If the authentications are failures, then the failure reason codes may also be useful.

`Logon Process` and `Authentication Package` can also provide useful information about how the authentication was attempted. If available, the `Process Information` can also provide valuable information about what specific process was responsible for attempting the authentication. A useful resource about the information available in logon event logs is available [here](https://www.ultimatewindowssecurity.com/securitylog/book/page.aspx?spid=chapter5#LogonEvents).

## Other Responses

### Notes From Other Accounts

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">First and foremost, I would consult my CMDB record. Is it a normal user workstation or a specific workstation installed with a legit tool that works by crawling and authenticating on all other workstation on the network. Ask the product owner to check if the tool behaves that way</p>&mdash; Daniel (@DanielOfService) <a href="https://twitter.com/DanielOfService/status/1590019965121630208?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">If it&#39;s a normal user workstation, continue with pulling security.evtx to search event 4624 and 4625 and identify the process that made the autentication requests to all other hosts. Pull other necessary evidence if necessary to investigate information surrounding the process</p>&mdash; Daniel (@DanielOfService) <a href="https://twitter.com/DanielOfService/status/1590020842335137794?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">For example, it may make sense to pull full memory capture to investigate the parent process and the command line argument of the responsible process</p>&mdash; Daniel (@DanielOfService) <a href="https://twitter.com/DanielOfService/status/1590021323920924673?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">- Understand auth in use. <br>- Understand logon session of that system (interactive/remote). <br>- 4776 (source=workstation name)<br>- 4624/25 ( RemoteIP=workstation ip)<br>- trend analysis on timestamps to see if its scripted. pull autoruns, prefetch, procmon of the system.</p>&mdash; Lionel F (@sandmaxprime) <a href="https://twitter.com/sandmaxprime/status/1590074167244001280?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-dnt="true"><p lang="en" dir="ltr">Also, reset the creds of the account that attempted the auth. <br>Check the LoginTypes of the events</p>&mdash; Lionel F (@sandmaxprime) <a href="https://twitter.com/sandmaxprime/status/1590074657742675968?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">- Source and Destination IPs<br>- Which Process and Port is responsible <br>- Which User<br>- Any connections from External to Source Computer<br>- Suspicious Port Connections <br>- Which Commands executed in the past and after the event<br>- Which usernames where used on the destination machines</p>&mdash; P3RPL3X_x25 (@P3rpl3xX25) <a href="https://twitter.com/P3rpl3xX25/status/1590014837840842753?ref_src=twsrc%5Etfw">November 8, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Notes From Chris

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Response of the week goes to <a href="https://twitter.com/DanielOfService?ref_src=twsrc%5Etfw">@DanielofService</a>. <br><br>When available, knowing the expected system role is helpful and sets the context for the next things you&#39;ll look for (like the process responsible for the activity). It&#39;s also easy to answer.<a href="https://t.co/8S4pP6oPZL">https://t.co/8S4pP6oPZL</a></p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1591440860901769218?ref_src=twsrc%5Etfw">November 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Many good responses -- lots of folks want to find the source process, which when examined, will reveal a lot regarding disposition. Many want to understand the ratio of success/failed logins. That may not help with disposition, but if malicious, will help with affected scope.</p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1591441508057903105?ref_src=twsrc%5Etfw">November 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">This example and the responses demonstrate how interpreting evidence can provide different types of cues. Dispositional cues hint at whether something is malicious or benign, and relational cues hint at the presence of additional relationships relevant to the investigation.</p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1591441716329979906?ref_src=twsrc%5Etfw">November 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">When possible, it&#39;s often more important to focus on the dispositional cues rather than the relational clues for the sake of expediency at the early stages of an investigation. Some relationships don&#39;t matter if you can quickly prove a benign disposition.</p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1591442162935156736?ref_src=twsrc%5Etfw">November 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">As always, a few paths with some leading to the same place. If you played along, pay special attention to how your knowledge of specific evidence sources influenced your choices and the cues you&#39;re able to identify or puruse. <a href="https://twitter.com/hashtag/InvestigationPath?src=hash&amp;ref_src=twsrc%5Etfw">#InvestigationPath</a></p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1591442714867834885?ref_src=twsrc%5Etfw">November 12, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Further down, Chris does provide some thoughts on where this information may have come from, stating:

<blockquote class="twitter-tweet" data-conversation="none" data-dnt="true"><p lang="en" dir="ltr">Likely routes are probably:<br><br>- SIEM rules triggering from Windows Event logs<br>- Threat hunting through logs looking for one-to-many relationships or large groups of success/files in a short time<br>- Observations from Windows logs made during an IR when assessing a suspect system.</p>&mdash; Chris Sanders 🔎 🧠 (@chrissanders88) <a href="https://twitter.com/chrissanders88/status/1590344588593594370?ref_src=twsrc%5Etfw">November 9, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Resources

- [4624: An account was successfully logged on](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624)
- [4625: An account failed to log on](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624)
- [Logon Events](https://www.ultimatewindowssecurity.com/securitylog/book/page.aspx?spid=chapter5#LogonEvents)
