---
layout: post
title:  "Bypass login pages"
date:   2018-05-29 18:05:55 +0300
image:  login.png
tags:   [101, bypass]
---

There are many different login pages. There are multiple ways to bypass them. so for the first one:

<h2>Try basic passwords</h2>

So basic yet works so great. But, don’t do it mindlessly. 

<h2>Routers</h2>
People usually does not change the passwords of there routers. We can easily find list of default user passwords for routers.


https://www.routerpasswords.com/

https://bestvpn.org/default-router-passwords/


So how to check the model of the router? First, probably the SSID of the router will be the same as the model number. 

Another trickier way is to check the MAC address. The OUI of the MAC is the first 24bit number (usually the first hexa number) which identifies the vendor or the manufacturer. For example:

00:14:78     TP-LINK TECHNOLOGIES CO.,LTD.

The MAC is really rarely changed so you can get an idea of which type of router  you are up against. You can check the MAC right here:

https://aruljohn.com/mac.pl

<h2>Getting Username</h2>

So from now on we are going to focus on webpages. Usually a login page needs a username/email and password. But how to find usernames/emails? So in this scenario we are on the login page  and we want to find out a user that we can use. 

<h3>Error</h3>

The easiest one is to start to try them. Random usernames. If you are lucky, the page error message will tell you: "Wrong username". If the username is right, it will change to something like "wrong password". That simple. You can find this on older Wordpress sites.

<h3>Forgot Password?</h3>

Next you can just visit the forgot password page. If you type a username or email there, if they are not valid, then the site will tell you "we could not find a user with this name". It is an older trick but you still can find it these days.

<h3>Register</h3>

So forgot password did not leak usernames. But there is a register page. We can try to register a username/email. If the username/email is already in the database, then we will probably get a "*already used" error.

<h3>Remember remember … The valid username?</h3>

Simple little trick to watch out for. A few sites will auto-fill valid usernames but not invalid ones. When you try a valid username and after submitting, it will appear in the username field, but not with a big random string, then you got a way to enumerate the users.

<h3>Time</h3>

Here is a trickier one. Usernames are hopefully hased these days. Hashing should take time. So developers can implement some performance improvement ideas like: "what if we only hash the user password when the username is valid. You don't need to check password when username is invalid right?" Yeeh, and NO. You can measure the difference in response times. So if the response time is longer than the average, you must have a valid username. Now let's look at how to do it with burp.

--burp tutorial--

<h2>Getting Password</h2>

So we got some usernames. People will not really change their usernames. Now let's go for passwords. The easiest way is just to:

-- use the force picture--

We have password list but now let's create our own. Let's say that we have one user we target. We gathered info about the target in a nice document (OSINT post coming out soon). We can just go through that document and make a password list from it. We can also visit the target organisation website and make a password list with tools like cewl.

--cewl code and hydra code--

<h3>Too many attempt</h3>

Oh that is bad. We got blocked after 3-5 attempts. So Bruteforce will not work? Let's think a little. We have a lot of usernames right? With a nice little password list. When we try for example Bob login 5 times, we get too many attempt time ban. What if we try one password for Bob, then the same password for John. If we are lucky, when we get back to Bob, the bad password counter will reset. 

--code--

<h1>prologue</h1>

We looked at so many ways to get in behind the login page. We have a lot more to cover so stay tune for the second part.
