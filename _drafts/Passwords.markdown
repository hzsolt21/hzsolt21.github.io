---
layout: post
title:  "Pass Words?"
date:   2019-06-04 18:05:55 +0100
image:  commic.png
tags:   [Hardware]
---

<link href='https://fonts.googleapis.com/css?family=Verdana' rel='stylesheet'>
<h1 style="font-family:Verdana">Hidden wifi hacking machine experience </h1>

<h2>Prologue:</h2>
There are always problems with passwords. How complex should it be, how big it needs to be. You have a lot of account and you cannot remember a unique password for them. You need to change passwords every 3-6 months, it is crazy. So let's look at what can be done.

<h2>Create Password</h2>
Let's create some password. Start with a capital letter. Let it be small no more than 8-10 char. Use a simple word connected to you, like your dog's name. The last two char should be a number with increment at every password change. Bumm most people password in a company. 

Fluffy12

Nice huh? The funny thing is that people won't increment by 2 or 3, they won't decrement the numbers at the end. Just i++ ;) 

That is not too secure. Let's make this bigger, more complex, and remember it. What if we don't use words… but sentences? Like: 

Sun Is Shining Pretty Good Today 

Let's add some number between words: 

Sun 12 Is 12 Shining 12 Pretty 12 Good 12 Today 12

Wow that is big and hard to brute force but easy to remember. If we know another language wi can switch a word. Like Good is Spanish: buena DECREMENTING the numbers and we got:

Sun 12 Is 11 Shining 10 Pretty 9 Buena 8 Today 7

Sure if someone knows the algorithm then you will have some problems, but this is a good start. Don't worry, it will be fast to type. Now let's check one password manager for each type. Remember there are many more.

 <h2>Password Managers:</h2>

A much better thing to do is to use a password manager. They generate passwords to you, they remember it you need just to copy past them. I usually separate 3 types of password managers. 

<h4>No storage</h4>
These type of password managers don't store passwords. They will generate the same password from the same input. They are the most secure ones in my opinion. The only problem is that for every password you need to remember different things. There is no error message, the password will be generated you need to try it to tell if you generated the right password or not.

<h4>Local storage</h4>
These type of password managers stores the passwords locally in an encrypted file The good side is that you only need to remember one thing. You do not depend on the vendors servers. The bad side is that you need to have the password file with you every time you need it. They can be stolen and brute forced so you need to have a strong password encryption for them. If you lose the file or the hardware dies and you don't have backup then you lost all your passwords.

<h4>Cloud storage</h4>
Same as the local storage ones but the company providing the program will store your password. Good side is that you will not have to have the password file with you every time. You can have your password every time everywhere. The bad side is that the company may be able to see your passwords. You depend on their servers and availability to have your password back. If they have a breach then you can have a breach. 

<h3>MasterPassword</h3>
This program is mostly in category No1. It will not store password anywhere. You can generate it on the fly everywhere anywhere.  Let's take a look:


--site http://www.masterpasswordapp.com/

You can download the app for IOS Android.. now I will just use the web. They ask for identity. You can enter anything, but remember them. This is how this app generates the pass for you. I entered: strik1r strik1r and used v3.

--picture masterpass

On the next page you can enter which site/app you want a password for like: google, facebook, wifi. Then the type of the password and the number of iterations. If you entered the same as me you should see the same result.

--masterpass pass

If everything's the same: username pass version sitename iteration passtype then the generated password will be the same.

<h3>Keepass</h3>
Keepas is an open source free password manager which stores the password in your local filesystem. It needs only a master password or a keyfile. It has a lot of unofficial ports including android and iphone ones.  

link: https://keepass.info/

Looks and feels like a database manager app. Because it is basically that. Works great as it should.

<h3>Lastpass</h3>
Lastpass is a cloud based password manager. You depend on them to get your password anytime anywhere. The support most browser and most devices. With there plugins, you can have autofill. This is the fastest way to enter the password. You can generate any number of characters with the password generator in any complexity. Here you can have MFA (multi factor authentication) to secure your password even better. There is a free version which is enough for the average user. Some problems can appear when reading their privacy statement at the current time:
"We may update this Privacy Policy to reflect changes to our information practices. If we make any material changes, we will provide notice on this website, and we may notify you by email (sent to the email address specified in your account), prior to the change becoming effective. "
That part of WE MAY is not sounding great. But is this a problem? Not really. Checking other companies, most of them have these tricky legal terms and conditions. So if you have a password manager that save to cloud, you can be sure that if the server goes down and your password is lost, they are covered. All in all lastpass is a great password manager for average people. Easy to use, free version, lots of support. Plus they claim to have zero knowledge which is a must. All these options are good enough? Sure, but let's get paranoid.

link:https://www.lastpass.com

<h2>Level Paranoid</h2>
Ok so password managers are really good and makes our life easier. But we want to get more secure. We don't want to instantly lose access in case out password manager is compromised. 

<h3>Extend</h3>
First thing we can do, is to add a little something to the password generated by the password manager. Kinda like salting. There are a lot of ways to extend the password. The easiest is to add the site name after the password. For example masterpassword using str1ker like above will get the password rexc pef rinkime gog for google. Adding a little extra, this case the site name and I have the password :rexc pef rinkime gog google or google rexc pef rinkime gog You can add sitename or username or your little pasword too. Same with every type of password manager. 



<h3>Merge</h3>
This is a good concept. Use two or more password manager to create one password. By that I mean something like create the first half of the password with master password, the second with Keepass. Basically the "salt" will be generated with another password manager. Although this is a really good and secure option, this can be complex and have multiple points of failure. Let's see:
Generate a password with masterpassword: rexc pef rinkime gog

Password generated with Keepass: X*SqDJAq>9HR

Final result: X*SqDJAq>9HRrexc pef rinkime gog  or rexc pef rinkime gogX*SqDJAq>9HR

What if we make it a little bit easier and generate a password to log in to the other password manager which will create the password for the desired site? That would basically reduce the complexity level. We would basically use one password manager at the end. 

 
<h1>Usernames</h1>
We are so obsessed with passwords. We are creating apps to manage them, creating devices for double factor authentication. What about usernames? Most people will find one or two username and that is it. It is much easier to find someone when the person uses the same username on every page. What can we do?

<h2>Generate</h2>
In case you use a good password manager, then you can simply generate a random username for different sites. We don't need to remember them. Let's See for example Master Password. We can use the site's name to generate username. Let's enter with the credentials above. Using googleu as sitename and Basic for the type, it will generate SNq48HFm. This will be my password. googlep and type phrase will give my password: ne holca caq hufejva
In case it is too complicated, we can always use an online generator like 

site https://jimpix.co.uk/words/username-generator.asp
<h2>Decoy</h2>
The best way, it is  to use someone else's username. Just get a lot of common username and  use a random generator to select one for different pages. Or better. Most username is a first name first letter + last name. Like John Anderson will be janderson. Bumm easy. So let's grab every common first name and every common lastname and pair the first letter of the first names with the last names and we are done.( btw for the first letters, use the English alphabet…)

https://en.wikipedia.org/wiki/Lists_of_most_common_surnames
https://en.wikipedia.org/wiki/List_of_most_popular_given_names


