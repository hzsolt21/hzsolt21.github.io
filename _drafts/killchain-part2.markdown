---
layout: post
title:  "Cyber Kill Chain - Part 2"
date:   2019-09-10 18:05:55 +0300
image:  CKC-2.png
tags:   [Blue Team,Kill Chain, Purple Team]
---


<h2>Prologue:</h2>
This is part 2 of Cyber Kill Chain series. In the previous blog posts we've talked about the initial 4 phases of the Cyber Kill Chain, in this blog post we will be discovering the remaining 3 phases of the kill chain. You can read my previous blog post <a href="https://redteamzone.com/part1-CyberKillChain/"> here </a>. We have discussed till the part where an attacker exploits some service / software in the network. The next phases of the kill chain talk about persistence, lateral movement aspects of an attack.


<h3> 5 - Installation:</h3>
<b>Attack</b> - In  this phase an attacker typically installs a backdoor into the system. Using this backdoor an attacker could persist a reboot / deletion etc within the exploited system.
An adversary needs to use persistence techniques in order to not lose access to the foothold he establishes in the network. The following are a few techniques which an adversary can use to gain persistent access on a system. 

Persistence Techniques: 
* Install a web shell 
* Modify registry keys to run on Startup
* Use AutoRun registry keys
* Install a new scheduled task (schtasks.exe, at.exe, CronJobs on Linux)
* Persist within Startup Folder / User Profiles (bashrc / bashprofile on Linux)
* Install backdoor as a service
* Add a new user account
* DLL Hijacking techniques etc. 


<b>Defense</b> - For a defender, this phase is important in organization's perspective. There are very few techniques for persistence and monitoring for changes related to those techniques. Few techniques defenders implement to detect attacker's activity:
* Monitor changes to User Profiles, Scheduled Tasks created etc.
* Install an EDR on the hsot to detect any anomalies.
* Monitor for files created and changes made to registry.


<h3> 6 - Command and Control: </h3>
<b>Attack</b> - An attacker needs his payload to callback to him every few mins or hours depending on the stealth he needs to maintain. In order to achieve that, attackers setup a command and control server which listens for incoming connections from their payloads. As the name suggests, the Command and Control servers are used to send commands and recieve the output as well. Multiple C2 servers are setup during an attack. Attackers use multiple techniques to hide their C2 servers from being detected. Attackers use multiple protocols / applications to get connections back to their C2 network. Few of those protocols are:
* HTTP
* HTTPS
* DNS
* SMTP
* FTP
* ICMP (extreme cases)
* SMB

There are so many frameworks / tools attackers use as C2 frameworks:
* Cobalt Strike
* Powershell Empire
* Covenant
* SlackOr etc.

<b> Defense </b> - In this phase defender's need to implement multiple layered protection based on the size of their network to detect outgoing C2 connections. Few techiques defenders use to detect are:
* Signature based detections on packet level over network
* Install a network proxy and scan for them
* Whitelist domains for outgoing traffic.
* Install root certificates on all the endpoints to monitor Host Headers. (Detect Domain Fronting)
* Install a firewall and configure ACLs

<h3> 7 - Missions on Objectives: </h3>
<b> Attack </b> - The last phase of an attack is where the attacker has everything in place and needs to just execute his mission to achieve the goals. Attackers don't have one single motive, they're driven by multiple actors. Few of their goals could be:
* Pubic defamation
* Espionage 
* Steal critical data
* Lateral movement within the network
* Modify / Overwrite data
* Disrupt the organization's operations
* Destroy systems or applications 
* Use the compromised environment for DDOS
* Demand Ransom


<b> Defense </b> - If an attacker came so far till this phase of the attack, that can only mean one thing, the defense failed big time. There were not enough security measures and mechanizsms in place. At this point of an attack defenders, usually have very few options left which are:
* Incident Response
* Post compromise assessment
* Detect unauthorized credential usage, data exfiltration etc
* Install Data Loss Prevention (DLPs) on endpoints to prevent any loss of critical data
* Conduct risk and loss assessment after the compromise 


<h2> Epilogue </h2> 
In this series of blogposts we learnt about the phases of an attack cycle and got an idea of few techniques used by an attacker. In upcoming blog posts we will be talking about many other techniques used by real world attackers and analyze few of them. We will attribute attacks using the kill chain and diamond model in upcoming blog posts. Happy reading!
