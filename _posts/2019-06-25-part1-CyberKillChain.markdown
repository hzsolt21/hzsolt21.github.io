---
layout: post
title:  "Cyber Kill Chain - Part 1"
date:   2018-05-29 18:05:55 +0300
image:  CKC.png
tags:   [Blue Team,Kill Chain, Purple Team]
---


<h2>Prologue:</h2>
This is part 1 of Cyber Kill Chain series. In this series of blog posts I'll be talking about various phases of an attack. With this series of Blog posts I hope to help people understand how blue team works and how threat intelligence can be gained by attributing the attackers based on various parameters. We will be exploring those parameters as well. 

<h2>Kill Chain</h2>
So what is this kill chain? Why do you need to even learn this? What benefit is it to know about something theoritical? These are the questions I commonly encounter. During the initial days of my exploration of the Blue side of security. I was wondering the same. Let's begin by learning what Kill Chain is.

<b>Kill Chain:</b> The Cyber Kill Chain framework is a model for identification and prevention of cyber attacks. It maps what steps the adversary (attacker) must have taken in order to achieve their goal / objective. This framework provides greater understanding of the TTPs, (Tactics, Tools and Procedures) used by the adversaries to decrease its chances/ desired outcome.

That is the proper definition of a Kill Chain. There are 7 steps of a cyber kill chain. Each step describes the phase of an attack and the steps taken by the attacker in that particular phase to achieve the goal. Here is the list of all the steps of a cyber kill chain. I'll be describing the phase in both the Attacker's and Defender's point of view so that we could learn the importance of that phase in both the perspectives. 


<h3> 1 - Reconnaissance:</h3>
<b>Attack</b> - The adversary/attacker gathers information on the target before the actual attack starts, this might include publicly available information on the internet, or if the attacker has some access, he might be looking for the software that are installed in the target system, test for  the vulnerable versions, look for open ports, and many more. Few things an attacker would like to get hands on are the Software in use 

Eg: 
* Which Operating System in use?
* Which Word Processor in use? 
* Which Browser is in use?
* Does the organization use any VPN / Proxy?
* Names or email addresses of employees for phishing etc.

So it basically means the attack may involve one or more of the following techniques: 
* OSINT (Open Source Intelligence)
* Passive or Active Scanning
* Social Engineering etc.


<b>Defense</b> - In defender's perspective there are few things that we could do to prevent Reconnaissance of an organization. Few of the steps involve:
* Decrease internet footprint
* Implement Firewall ACL (Access Control Lists) rules for incoming traffic
* Implmenet Web Analytics to analyze the traffic (Eg: Google Analytics)


<h3> 2 - Weaponization: </h3>
<b>Attack</b> - The attacker uses the gathered information about the target to develop exploits or malicious payloads to send the victim. All the information gathered from the previous phase including the software / hardware in use related information will be analyzed and few of the following things can be done by the attacker:
* Make a Document with a malicious Macro
* Design a phishing mail 
* Make a malicious PDF file
* Prepare USB drives for a driveby attack etc.


<b> Defense </b>- In defense perspective there is nothing a defender could do as this is not an active phase. The attacker doesn't directly interact with the target during this phase. Implementing proper defenses to prevent reconnaissance can stop attacker from being successful in this phase.

<h3> 3 - Delivery: </h3>
<b> Attack </b> - This phase involves attacker sending the malicious payload to the victim by e-mail or other means. Delivery methods of attacker's vary usually depending on the organization's weakest link. Humans are the weakest link to an organization usually. So the following are the common delivery methods used by attackers:
* Phishing Email
* Email with Malicious Attachments (ISO, PDF, WORD File with Macros, HTAs etc.)
* Drop few infected USB drives in a public place etc.


<b> Defense </b> - There are multiple ways to stop this delivery by attackers. First and foremost important thing is to make the employees aware of the cyber attacks.
A vigilant user should be able to detect malicious payloads sent by attackers. Few of the defenses for this phase are:
* Implement proxy filter to deny outgoing traffic
* Implement an Inline EDR or Anti Virus solution 
* Use email sandbox technologies
* Use Application Filtering Firewall.

<h3> 4 - Exploitation: </h3>
<b> Attack </b> - During the exploitation phase, the APT's malware code is executed on the target network through remote or local mechanisms, taking advantage of discovered vulnerabilities to gain superuser access to the targeted organizational information system. Attacker's in this phase try to escalate privileges from a normal user to a higher privileges such as Administrator, Root User etc. Attackers usually use sophisticated techniques or sometimes 0day's to gain access to privilged accounts. 

<b> Defense </b> - Defender's should always keep the systems updated by applying vendor's security patches wherever applicable. This can hinder an attacker's exploitation attack. 
* Use EDR solutions to detect and stop 
* Implement "chroot" jail on the hosts facing internet
* Use anti exploitation techniques such as EMET, DEP etc.
* Keep the systems and applications upto date with all patches 


<h2> Epilogue </h2> 
In this blog post we discovered what a kill chain is and what are the phases of the kill chain. We explored a bit about first 4 phases of the cyber kill chain. The recon, weaponization, delivery and exploitation phases of the kill chain. We also learnt few defenses for the respective phases. In part 2 of this series let us explore more about the next 3 interesting phases of the kill chain. 