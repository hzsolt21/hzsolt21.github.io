---
layout: post
title:  "Login Page - Bypasses"
date:   2018-05-29 18:05:55 +0300
image:  login.png
tags:   [101, bypass, web]
---

There are many different login pages. There are multiple ways to bypass them, so for the first one:

<h2>Basic passwords</h2>

So basic yet works so great. But, don’t do it mindlessly. 

<h2>Routers</h2>
People usually does not change the passwords of there routers. We can easily find list of default user passwords for routers in the links below.

[Router default password list](https://www.routerpasswords.com/)

[Router default password list 2](https://bestvpn.org/default-router-passwords/)

So how to check the model of the router? First, probably the SSID of the router will be the same as the model number. 

Another trickier way is to check the MAC address. The OUI of the MAC is the first 24bit number (usually the first 3 hex bytes) which identifies the vendor or the manufacturer of that specific router. For example:

00:14:78     TP-LINK TECHNOLOGIES CO.,LTD.

The MAC is really rarely changed so you can get an idea of which type of router  you are up against. You can check the details of the MAC address and their vendors right here:

[Mac address resolver](https://aruljohn.com/mac.pl)

<h2>Username Enumeration</h2>

So from now on we are going to focus on webpages. Usually a login page needs a username/email and password. But how to find usernames/emails? So in this scenario we are on the login page and we want to find out a username or email address that we can use. 

<h3>Enumerating via Error Pages</h3>

The easiest one is to start to try them. Random usernames. If you are lucky, the page error message will tell you: "Wrong username". If the username is right, it will change to something like "wrong password". That simple. You can find this on older Wordpress sites.

<h3>Enumerating via Forgot Password?</h3>

Next you can just visit the forgot password page. If you type a username or email there, if they are not valid, then the site will tell you "we could not find a user with this name". It is an older trick but you still can find it these days.

<h3>Register</h3>

So forgot password did not leak usernames. But there is a register page. We can try to register a username/email. If the username/email is already in the database, then we will probably get a "*username already in use*" error.

<h3>Remember remember … The valid username?</h3>

Simple little trick to watch out for. A few sites will auto-fill valid usernames but not invalid ones. When you try a valid username and after submitting, it will appear in the username field, but not with a big random string, then you got a way to enumerate the users.

<h3>Time</h3>

Here is a trickier one. Usernames are hopefully hashed these days. Hashing should take time. So developers can implement some performance improvement ideas like: "what if we only hash the user password when the username is valid. You don't need to check password when username is invalid right?" Yeah, and NO. You can measure the difference in response times. So if the response time is longer than the average, you must have a valid username. You can try this with Burp or Owasp Zap.

--burp tutorial--

<h2>Getting Password</h2>

So, now that we got some usernames and usernames are not changed any frequently, let's go for passwords. The easiest way is just to bruteforce the passwords:

-- use the force picture--

We have password list but now let's create our own. Let's say that we have one user we target. We gathered information about the target in a nice document (OSINT post coming out soon). We can just go through that document and make a password list from it. We can also visit the target organisation website and make a password list with tools like CEWL.I will not write down the usage of cewl, hydra, medusa, hashcat and may more password cracking tools, but I will leave a link for most of them. 

[cewl](https://tools.kali.org/password-attacks/cewl)
[hashcat](https://tools.kali.org/password-attacks/hashcat)
[hydra](https://tools.kali.org/password-attacks/hydra)


<h3>Too many attempts</h3>

Oh that is bad. We got blocked after 3-5 attempts. So Bruteforce will not work? Let's think a little. We have a lot of usernames right? With a nice little password list. When we try for example Bob login 5 times, we get too many attempt time ban. What if we try one password for Bob, then the same password for John. If we are lucky, when we get back to Bob, the bad password counter will reset. This technique is also called as Password Spraying. 

<h1>prologue</h1>

In this part we had a look at many ways to get in behind the login page. This post has already explained it's length and so, the next part will cover the rest of the techniques necessary to bypass the login page.
