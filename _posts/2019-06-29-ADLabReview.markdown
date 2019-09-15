---
layout: post
title:  "My Active Directory Lab Experience"
date:   2019-06-27 18:05:55 +0100
image:  ADLabReview/ADLabReviewHeader.png
author-name: jinxbox
tags:   [Hardware]
---

<link href='https://fonts.googleapis.com/css?family=Verdana' rel='stylesheet'>
<!-- <h1 style="font-family:Verdana">My Active Directory Lab Experience </h1> -->

<h2>Prologue:</h2>
Back in February 2019, I wanted to dive in deeper into the active directory aspects of security. I just had a minimalistic idea about it, but I always wanted to learn the attacks on Active Directory from both the Red and Blue team's perspective. I saw the tweet from PentesterAcademy about the new course they were going to launch "ACTIVE DIRECTORY LAB". I just saw the content covered in the course and the price of the course. Immediately, I WAS ALL IN! I might be one of the first few ones who signed up for the course. I purchased the 30 - days lab time. I just wanted to share with you about what I feel about the Active Directory Lab from PentesterAcademy.  

<h2>About the Course</h2>
Starting off, the course covers mostly everything you need to begin active directory pentesting or red team recon in an active directory environment. Few of the things the course covered are:

* Active Directory Enumeration
* Local Privilege Escalation
* Domain Privilege Escalation using Kerberoast, Kerberos delegation, Abusing protected groups, abusing enterprise applications and more. 
* Domain Persistence and Dominance using Golden and Silver ticket, Skeleton key, DSRM abuse, AdminSDHolder, DCSync, ACLs abuse, host security descriptors and more. 
* Forest privilege escalation using cross trust attacks. 
* Inter-forest trust attacks

Course basically starts from what an active directory actually is and ends with techniques to perform domain privilege escalation and cross-forest trust attacks. Sounds Awesome right? Yes, because the course truly delivers the content written on their website. This course is more based on misconfigurations than exploitation. That being said, I feel that is the real fun about this course. That is essentially how it is different from other courses/labs. 


<h2>About the Labs</h2>
The labs provided were Windows 2016 servers with everything set. You will be provided an RDP access to a machine which is part of the Domain you'll be hacking into. The labs are super stable. I've never faced an issue while I was doing the lab. This course follows a similiar approach to the "Assume Breach" methodology. It is assumed that the attacker is already inside the Domain/Network. This lab is highly learning objective based meaning there are 23 learning objectives for you to finish before your lab time ends.


![Lab Overview](/img/ADLabReview/activedirectorylab.png) 

<h5>Few of the tools I've used during the labs are: </h5>

* [Mimikatz](https://github.com/gentilkiwi/mimikatz/tree/master/mimikatz)
* [Powersploit](https://github.com/PowerShellMafia/PowerSploit)
* [HeidiSQL - Portable](https://www.heidisql.com/download.php)
* [Powercat](https://github.com/besimorhino/powercat)
* [Kekeo](https://github.com/gentilkiwi/kekeo)
* [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)
* [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation)
* [Nishang](https://github.com/samratashok/nishang)


<h2>About the Instructor</h2>
Nikhil Mittal is the instructor of this course and that's a real bonus. His explanations were on point and crystal clear. He managed to make the the course objectives as well as learning objectives of the lab as easily as understandable and as doable as possible. Special mention to the design of the course where he shows not only the offensive side of the attack but also the defensive side of the attack. This adds HUGE value to the course. Few places he shows the misconfiguration from the GUI of the Domain Controller. These subtle things are really important if you're into consulting etc. where you need to provide actual recommendations to your clients. Or if you're a sysadmin this portion of the lab is highly useful. I'm pretty sure Nikhil kept this in mind for the course. 


<h2>and then THE EXAM.. </h2>
It is a 24-hour exam, with additional 48-hours to make the report and submit. There are few requirements for the report. I finished the exam in around 3-4 hours and made the report the next day. Next, within 24 hours got a mail from them about me clearing the exam. Also got a personal mail from Nikhil Mittal, he was open to any feedback and improvements that could be done to the lab/exam. 

![Exam Certification](/img/ADLabReview/ADLabCert.png)

Few exam conditions that will have to be met for the certification are:

* Report should be professional! 
* Report should include screenshots of tools used and their output
* Report should consist proper recommendations to the issues you've identified in the exam.

Overall, the exam is not as difficult as you might think it is, unless you want to complicate it so much. There are a few rabbit holes in which you could deeply fall into. My suggestion for the exam is to keep it as simple as possible :).



<h2>So, what's next?</h2> 
I really enjoyed taking the course and never regretted even for a second. I was really happy to have a playground in which multiple things could be tested out as well. I will probably take their RED TEAM LAB course which is a challenge lab and not a learning lab as this one. That lab is more tougher than the Active Directory Lab and will also be having more challenges than the current Active Directory Lab. I've heard really good things about the Red Team Lab as well. 

<h2> Few other AD Playgrounds to get your hands dirty </h2>


* XEN from HackTheBox
* EndGame from HackTheBox
* RastaLabs hosted on HackTheBox
* OffShore lab hosted on HackTheBox 