---
layout: post
title:  "Brute-Force"
date:   2018-05-29 18:05:55 +0300
image:  BF.png
tags:   [Hardware]
---
<h1>Brute Force 433 signal</h1>

<h2>Prologue</h2>

We can now catch and replay the code from a single remote control. That is nice but here is the problem. What if we want quick access to something? Can we bruteforce the code? That is what I will try. The code that I got last time (15588040) indicate that we have a lot of combination to go trough. 

<h2>First try</h2>
Before we try, we need to know how much time we will need to wait between signals. Can we send them after one another or should we use a delay that will drastically slow us down. For that I will modify the sender code a little bit. instead of sending 15588040, i will send 15588039 15588040 15588041 after each other without wait. If I can observe a change then I will know that no delay is needed.

--code---

Succes, that will make our lives a little bit easier. Let's see how much time does it take to send 100 command. For that I will use a  for loop and send a random number. Now I will not check if a change happens, I only go for speed.

-- code --

As we can see the speed is dramatically low. With this speed it would take ---time -- to hack a door. Not a valid option. 

<h2>We need to go faster</h2>

So we need something tricky. But if we take a look at this number: 15588040 it is not just a number. it has 1, 5, 8 , 0 ,4 , 40 and all these numbers inside it. So If our code is 15588040, will it activate for 115588040 or 222215588040? We could send a few big sequences of numbers that would contain most of the numbers. Even if our key is 8 digits like: 15588040 then 115588040 would be 2 key at once. it would be 11558804 and 1558804. So let's try it.

--code--

Sadly it did not work. But can we go even further?

<h2>Go binary</h2>

So  basically we are sending decimal numbers. But in the backend these numbers are converted to signals. Those contains only 0 and 1. Let's look at the binary representation of 1558804. Quickly converting it we are getting the number: 101111100100100010100

