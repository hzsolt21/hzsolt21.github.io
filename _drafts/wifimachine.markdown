---
layout: post
title:  "Hidden machine"
date:   2019-06-04 18:05:55 +0100
image:  433MhzRemote-2.png
tags:   [Hardware]
---

<link href='https://fonts.googleapis.com/css?family=Verdana' rel='stylesheet'>
<h1 style="font-family:Verdana">Hidden wifi hacking machine experience </h1>

<h2>CAUTION:</h2>
This project can be dangerous. I am not responsible for any damage or injury. This is a prototype project. Doing this is not recommended. Do it on your own risk. 
<h2>CAUTION</h2>

<h2>Prologue:</h2>
Everyone uses wifi. When you want to get inside a network, wifi is really good because you can be outside of the target. Just sit down on a branch and get inside the network. Or grab some packets for later wifi password cracking. Usually you need a laptop for that. Using a wifi adapter to set it in monitor mode and grab every packet. This can be suspicious. They look back at the photage of the outside security cameras and they can spot you. Or a worker see you wit a weird gadgets near the office. Now I want to find a way to do this and be invisible. At least harder to find. For this, I will use my phone. Today everyone uses their phone. Someone typing on it will not be a big deal. People type on their phone even when they are driving. Phone wifis usually can't go into monitor mode. You need special hardware for it.  Once again, it is weird and unusual to use a phone with a wifi adapter. So I was thinking on creating a portable single board computer and hide it. Let's look at what happened.

<h2>Let's Start:</h2>

Getting a Raspberry Pi or an Orange pi, setting up Kali and put a portable charger as PSU is not a big deal. Hiding it well will cause new problems. So here is my basic idea:

-Get a single board computer

-Set up Kali with a wifi adapter

-Put it in a backpack

-SSH in with a Phone

Seems simple, let's do it.

<h2>Orange</h2>

I chose an Orange Pi --exact model with link-- The main reason is, that this is a prototype, and I 
already have Kali up and running on it. You can use Raspberry Pi as well. I won't write a step by step on how to install Kali, here are some links if you need it:

[KALILINK](http://www.orangepi.org/downloadresources/)

Wifi adapters, just find the one that can be used in monitor mode. Up to you. I will use the main wifi on Orange Pi to connect to my phone. This way I can ssh in and do everything from my phone. I will use the wifi adapter for hacking.

<h2>Thermal</h2>

Here we go. the biggest problem. These computers can be hot. If you place it in your backpack then it is worst. I did a few measurements to check if I can put it in my backpack with a powerbank. First I created a little script that will log the thermals every 10 second. There is 2 file with the current values. These will be saved to 2 separate files: temp1 and temp2.

```
#!/bin/bash
while true; do
	cat /sys/devices/virtual/thermal/thermal_zone1/temp >> temp1;
	cat /sys/devices/virtual/thermal/thermal_zone0/temp >> temp2;
	sleep 10;
done
```

The tests consist of an idle and a password cracking test. I created a hash from a password which is not in rockyou.txt, and run john to crack it. Not the best stress test, but this machine will not do more performance hungry job then this. Creating the hash and running john with logger:

```
echo -n ASDGqw324qarfhaqzhaewrgf | sha1sum > hash
john hash --wordlist rockyou.txt & logger.sh &
```

Let's run logging in Idle on air first. No cooling just left it in the room. Nothing really interesting happened. The temperature stopped at 53C°. Not too bad. Let's make it interesting. Running john will bring the temperature up.. but not too much. 

![Air Temps](/img/wifiMachine/air.PNG)

This is good. But what if you put this in a backpack. I grabbed my backpack, put the orange pi and let's run it. Idle was not too interesting. Stopped at 65C°. Let's run John and see the result.

![Backpack Temps](/img/wifiMachine/backpack.PNG)

Here is the problem. I had to stop the experiment. 80C° started to get hot. I was afraid that something will catch fire. The backpack is a non-fireproof backpack. Something must be done with this. There are a number of ways to deal with the heat problem. I could cut holes, put in fans. I want something more portable, more crazy. I saw those PCs where they put the hole machine in mineral oil. It works really well because the oil is non conductive, non corrosive. The  whole computer becomes a mini submarine. My plane is a little bit different. I got a little food container which can take heat up to 110C°. Mineral oil is not something that is instantly available. So I grabbed some synthetic motor oil. It should be non conductive and non corrosive as well. I don't know what it can do in the long run, but I was sure that it will work. Let's make some test first. I did not want to put the Orange pi directly into the liquid. I made a simple circuit with an arduino nano All it does is to turn a led light on and off. I soldered them with a resistance and the test could begin. 

```
int led_pin=12;

void setup(){
  pinMode(led_pin,OUTPUT);

}

void loop(){
  digitalWrite(led_pin,HIGH);
  delay(500);
  digitalWrite(led_pin,LOW);
  delay(500);
}
```

![Arduino](/img/wifiMachine/arduino.jpg)

After a few hours the led still blinks. It is working so far. I could start the real test.

-- gif of pouring the oil

At first the idle results were epic. From current 63C° an instant drop to 42C°, and the best is, it did NOT EXPLODE. I had to cut some holes in the lid of the box for the power cable and the wifi extension cable. Those are sealed up with superglue and hot glue. Just to be sure. I closed the lid and sealed as well. Grab the backpack and can start the measurement. In idle nothing happens. Everything is at 42C°. Let's crack some password and see the result.

![Liquid Temps](/img/wifiMachine/liquid.PNG)



