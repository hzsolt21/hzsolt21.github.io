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
First, let's run it agains the target.
[FAIL](/img/EternalBlue/fail.png)
This did not go very well. No named Pipe was found. We can add more possible named pipe to our list, and give it another try. It failed again. From here we have another option. 
If we know valid credentials, then we could try to use it. Adding vagrant/vargrant to the Username Password section.
```USERNAME = 'vagrant'
PASSWORD = 'vagrant'```
Running the code it will execute and create C:\pwned.txt on the machine.
[PWNED](/img/EternalBlue/pwned.PNG)
We can log in and check the file. It should be there. That is ok but vagrant is an admin user. If we have credentials then we probably do not need this exploit at all.

<h3> Eternal Blue as Privilege escalation </h3>
Let's create a test user with test as password. User test should not be admin this time. By loging in as test, you can see that you are unable to create c:\pwned.txt.
[testUser](/img/EternalBlue/test.PNG)
Adding these credentials to our exploit and running it, will once again create the file pwned.txt This means that we have non-admin credentials but we could still execute commands as admin.

<h4> Modifying the exploit</h4>

<br>


link[@WarMarx](https://twitter.com/\_WarMarX\_) 