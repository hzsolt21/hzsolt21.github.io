---
layout: post
title:  "Eternal Blue without Metasploit"
date:   2019-10-15 18:05:55 +0300
image:  EternalBlue/title.png
author-name: strik1r
tags:   [Eternal, Blue, Without Metasploit, Exploit modification]
---

<h2>Prologue:</h2>
I wanted to create a Without Metasploit series. Here we would look and modify famous exploits without using the Metasploit framework. What would be the best way to start then with Eternal Blue?
<h3> Environment </h3>
The easiest way to find an environment to test on, is to get Hack The Box VIP and attack Blue.
Setting up your lockal vm, try Metasploitable 3 [Metasploitable3](https://github.com/rapid7/metasploitable3) 
I will go with the Metasploitable now.
<h3> Vulnerability Description </h3>
ThinVNC uses Basic Authentication to authenticate a user to access the web VNC interface. Credentials to be used are set on the server side while deploying the VNC server. There is no fixed port on which VNC server runs, you can run the VNC server on any port pre-configured.

A sample authentication screen looks like this:
![auth](/img/ThinVNC/CVE-2.png)

When tried with multiple with wrong authentication credentials, it throws a HTTP 401 error. This can be bypassed using the following vector: <br>
<b>**/../../../../../../../../../windows/win.ini** </b>

<h4>Sample Request:</h4>

```
code
```
<br>
![Burp](/img/ThinVNC/CVE-4.png)
<br>

This directory traversal attack vector allows us to read any arbitrary file on the system. We can use the same vector to steal the credentials for the VNC client. Once we steal them, we can use the credentials to compromise the VNC server. Post compromise, we get terminal access to the VNC server.



<h3> POC: </h3>
link[@WarMarx](https://twitter.com/\_WarMarX\_) 