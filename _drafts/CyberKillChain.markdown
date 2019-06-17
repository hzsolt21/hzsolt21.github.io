---
layout: post
title:  "Cyber Kill Chain - Part 1"
date:   2018-05-29 18:05:55 +0300
image:  login.png
tags:   [Blue Team,Kill Chain, Purple Team]
---


<h2>Prologue:</h2>
This is part 1 of Cyber Kill Chain series. In this series of blog posts I'll be talking about various phases of an attack. With this series of Blog posts I hope to help people understand how blue team works and how threat intelligence can be gained by attributing the attackers based on various parameters. We will be exploring those parameters as well. 

<h2>Kill Chain</h2>
So what is this kill chain? Why do you need to even learn this? What benefit is it to know about something theoritical? These are the questions I commonly encounter. During the initial days of my exploration of the Blue side of security. I was wondering the same. Let's begin by learning what Kill Chain is.

Kill Chain: The Cyber Kill Chain framework is a model for identification and prevention of cyber attacks. It maps what steps the adversary (attacker) must have taken in order to achieve their goal / objective. This framework provides greater understanding of the TTPs, (Tactics, Tools and Procedures) used by the adversaries to decrease its chances/ desired outcome.

That is the proper definition of a Kill Chain. There are 7 steps of a cyber kill chain. Each step describes the phase of an attack and the steps taken by the attacker in that particular phase to achieve the goal. Here is the list of all the steps of a cyber kill chain. I'll be describing the phase in both the Attacker's and Defender's point of view so that we could learn the importance of that phase in both the perspectives. 


<h3> 1 - Reconnaissance:</h3>
Attack - The adversary/attacker gathers information on the target before the actual attack starts, this might include publicly available information on the internet, or if the attacker has some access, he might be looking for the software that are installed in the target system, test for  the vulnerable versions, look for open ports, and many more.

So it basically means it may involve: 
* OSINT (Open Source Intelligence)
* Passive or Active Scanning
* Social Engineering etc.


Defense - In defender's perspective there are few things that we could do to prevent Reconnaissance of an organization. Few of the steps involve:
* Decrease internet footprint
* Implement Firewall ACL (Access Control Lists) rules for incoming traffic
* Implmenet Web Analytics to analyze the traffic (Eg: Google Analytics)

