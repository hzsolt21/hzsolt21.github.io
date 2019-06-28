---
layout: post
title:  "Active Directory Lab Review"
date:   2019-06-04 18:05:55 +0100
image:  433MhzRemote-2.png
tags:   [Hardware]
---

<link href='https://fonts.googleapis.com/css?family=Verdana' rel='stylesheet'>
<h1 style="font-family:Verdana">My Active Directory Lab Experience </h1>

<h2>Prologue:</h2>
Back in February, I wanted to dive in deeper into the active directory aspects of security. I just had a minimalistic idea about it, but I always wanted to learn the attacks on Active Directory from both the Red and Blue team's perspective. I saw the tweet from PentesterAcademy about the new course they were going to launch "ACTIVE DIRECTORY LAB". I just saw the content covered in the course and the price of the course. Immediately, I WAS ALL IN! I might be one of the first few ones who signed up for the course. I purchased the 30 - days lab time. I just wanted to share with you about what Active Directory Lab did to me.  

<h2>About the Course</h2>
Starting off the course covers mostly everything you need to begin active directory pentesting or red team recon in an active directory environment. Few of the things the course covered are:

* Active Directory Enumeration
* Local Privilege Escalation
* Domain Privilege Escalation using Kerberoast, Kerberos delegation, Abusing protected groups, abusing enterprise applications and more. 
* Domain Persistence and Dominance using Golden and Silver ticket, Skeleton key, DSRM abuse, AdminSDHolder, DCSync, ACLs abuse, host security descriptors and more. 
* Forest privilege escalation using cross trust attacks. 
* Inter-forest trust attacks

Course basically starts from what an active directory actually is and ends with techniques to perform domain privilege escalation and cross-forest trust attacks. Sounds Awesome right? Yes, because the course truly delivers the content written on their website. 


<h2>About the Labs</h2>
The labs provided were Windows 2016 servers with everything set. You will be provided an RDP access to a machine which is part of the Domain you'll be hacking into. The labs are super stable. I've never faced an issue while I was doing the lab. This course follows a similiar approach to the "Assume Breach" methodology. It is assumed that the attacker is already inside the Domain/Network. This lab is highly learning objective based meaning there are 23 learning objectives for you to finish before your lab time ends. 

<-- ADLAB PIC --> 




<h2>So, what's next?</h2> 

I will probably take their RED TEAM LAB course which is a challenge and not a learning lab as this one. That lab is more tougher than the Active Directory Lab and will also 