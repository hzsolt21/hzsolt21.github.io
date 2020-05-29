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

Attacker
There are a few things you need to set up on your attacker machine too. First install pip, then impacket.
```
sudo apt-get update
sudo apt install python-pip
git clone https://github.com/SecureAuthCorp/impacket
cd impacket-master
pip install .
```
From here it should work just fine.
<h3> The Exploit </h3>
If you do a ```searchsploit eternalblue``` you will get the exploit 42315.py
![searchsploit](/img/EternalBlue/searchspolit.png)
Copy it to our directory ```searchsploit -m 42315.py```

Now let's start to look at what do we have in this file.

first we can see the lines:
```from impacket import smb, smbconnection
from mysmb import MYSMB```
We already have impacket but what about mysmb? With a little googleing we can find that too.
```wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py```

Now we can go deeper.

```USERNAME = ''
PASSWORD = ''```
We should change this if we find valid smb credentials. Going a little be down, there is a nicely named function find_named_pipe. This means that we
don't need to run other exploit/scanning to find our named pipes because this is already included in the exploit itself. However checking the first lines
of the function reveals the default pipes that the function will search for. ```pipes = [ 'browser', 'spoolss', 'netlogon', 'lsarpc', 'samr' ]``` you can expend
this list if you did not succed for the first try.


<h4>Sample Request:</h4>
try, fail then add username then show that low priv user can do it
```
code
```
<br>
![Burp](/img/EternalBlue/searchspolit.png)
<br>

This directory traversal attack vector allows us to read any arbitrary file on the system. We can use the same vector to steal the credentials for the VNC client. Once we steal them, we can use the credentials to compromise the VNC server. Post compromise, we get terminal access to the VNC server.



<h3> POC: </h3>
link[@WarMarx](https://twitter.com/\_WarMarX\_) 