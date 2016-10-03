---
layout: post
title:  "Powershell Shells"
date:   2016-10-03 15:31:39
categories: security shells powershell
---

It's been a while... I figured it's about time I post something here again.

A while back I was required to see how much damage can be done by a malicious staff member. The one caveat here was that I had to test directly from the Windows box and had extremely limited outbound comms. For various reasons the usual tool-suits were out and I took this as a challenge to see how much damage I could do by coding tools on the "employee machine".

I was able to get [Powerview](https://github.com/PowerShellMafia/PowerSploit) onto the box, which helped identify possible targets on the network. One of the targets identified was an instance of ManageEngine with default credentials. Browsing around this instance I quickly found that you could upload files or scripts to excute on the ManageEngine box. Natuarally I could do something simple like upload the commands I want to execute all contained within a single file. However I was in the mood for interactive access. All ports other than 8080 were firewalled off, but the ManageEngine box was able to talk back to the workstation under my control. This created a new issue, what to upload? Why not some powershell? For this I re-used a Reverse Shell in Powershell that I'd written a while back,
{% gist staaldraad/204928a6004e89553a8d3db0ce527fd5 %}

With my reverse shell on the target, I needed a listener to actually receive the connection. Usually I would use Netcat (nc) for this, however, I had no nc instance available to me. Powershell to the rescue, a few lines of Powershell later, we have a Listener in Powershell,
{% gist staaldraad/8473da7f2dfed28b2216b15ca6ebad11 %}

This allows us to "catch" an incomming shell and establish a two-way communication channel. This is rather crude and rudimentary but was sufficient in allowing me to fully compromise the ManageEngine box.

A rather simple solution but hopefully comes in use to someone in the future. In most cases you would probably get Powershell Empire set-up, for those times you don't, you now have a simple Powershell reverse shell and listener.
