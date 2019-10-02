---
layout: post
title:  "Hash Crypto Stego"
date:   2019-09-10 18:05:55 +0300
image:  Crypto/title.png
author-name: strik1r
tags:   [Crypto, Stego]
---


<h2>Prologue:</h2>
Let's create a scenario to make things easy. Let's say we have Bob and Alice. They want to send messages to each other. There is a hacker who can make there life hard. What can they do?


<h3>plain text</h3>
They can send messages in plain text. This is not a great idea. Everyone, like our hacker, can see what they are sending. He can send messages in the name of Bob or Alice. We can see that this is not good enough.

Good:
Fast, no extra computing power needed

Bad:
Unsecure

<h3>hash</h3>
Hashing is when you take a message, apply an algorithm and get a fix length of random characters. It does not matter what you has. It can be a letter, a file, anything. We will just call them messages. Any message hashed should give back the same length of characters. That is because the hash length would change, you could make statistical guesses to make it easier to get back the original information. Every hash should have a flat histogram. That means that the number of characters in a hash should be around equal. One of these well known algorithms is MD5. MD5 is a 128 bit hashing algorithm. But how can Bob and Alice use this to messages to each other? Not really well. To do it, both of them needs a list of predefined messages. 

For example, "Go and by milk" wil be ca127333b415573a22753d74f439c863.

Bob can get this hash, get his list and check which message will have the same hash. This is good because the hacker can't instantly tell, what the message was. The hacker is able to create a message, but  if it is not in the list, then neither Alive nor Bob can read it.

There is a problem with hashes. Every massage should have different hash. Since a hash length is fixed, no matter the algorithm, this is not possible. Let's prove that. In a small example. We have this MD5. It is 32 characters long. Let's say MaxNumber is the number of every possible combination of letters and numbers that can fit in 32 char. this will contain a hash with only the letters. Which will not be a valid MD5 hash because it hash a really big spike in the histogram. So our MaxNumber is bigger than the actual number of valid MD5 hashes. Let's hash every number until MaxNumber. If there was no two numbers with the same hash until then, then there was a so called hash collision. If there was no number with the same hus until here, then the next number must have a has that was already used, because there is no more hash to use.

So why do people use hashing if there is a collision? The Answer is simple. A modern hashing algorithm, like SHA3-512 has a hsah so long, with so many possibilities, that a collision is really unlikely to happen. 

Good:
Attacker can't see the messages
Attacker can't send messages in the name of someone else, if the messages list is unknown

Bad:
Hashing is not for secure communication.
Slow and only predefined messages can be sent.
 
<h3>Cryptography</h3>
Now we are going somewhere. Cryptography is the science of hidden messages.This is ment to send messages to one another. In easy way, crypto is:
have a message
get a key
make the message unreadable with the key
target will read the message with a key

Crypto messages does not need to have a fixed length, so no collision problem. It was intentionally written "a key" instead of "the key". Crypto can use one or two keys.

<h4>Pre-shared key</h4>
This is basically the one key crypto. Also called Symmetric Encryption. Think of it like this. Bob and Allice choose a key. Let's say it's "Password123". Not too secure but still something.  With this, the choused algorithm will encrypt the message. As long as Alice and Bob know the password, they can read and send messages to each other. This is how your wifi works. You have a password, as long as you have the password you can log in to the router. Perfect. This is what we wanted. But still there are problems. An attacker who knows the password, can read everything. Worst is, that if an attacker record the encrypted messages for a long time, then when they find out the password, they can decrypt all messages from the past. Not something we want. If an attacker finds out the password, they can send any message in the name of Bob or Alice. Because the attacker can decrypt the messages from the past, we cannot send a new password in the same channel. Alice and Bob need to find a way to constantly change the password.


<h4>Two key crypto</h4>
It is called Asymmetric Encryption. From the title it is easy to figure out what this actually is. With the help of some really nice mathematical algorithms, you can create encryptions with two key. (If you want to read more about this, search how RSA or ECC algorithms works) One is for encrypting the other is for decrypting. Here is how this works. Bob creates a pair of keys. He keeps one and sends one to Alice. Alice does the same. Alice will create a Symmetric Encryption key and encrypt it with the key from Bob. Sends her key and the encrypted key to Bob. Now Bob and only Bob can decrypt the Symmetric Encryption key. With this Symmetric Encryption key they can now send messages to each other. After some time, one of them can just change the key, encrypt it and send it to the other. 

PICTURE

But wait. If Asymmetric Encryption is so secure, then why don't we just send messages with it. Why do we need symmetric? Simple. Symmetric needs much less resources to do. Faster and use less power.


Good:
Crypto and the methods built on them is the go to messaging solution.
No collusion
Mathematically impossible to crack most of the algorithms

Bad:
Attacker can still see that you have encrypted communication which can make you suspicious. 

<h3>Steganography </h3>
To sum up steganography: Hidden Cryptography. The messages are hidden in a way that an observer cannot tell that you are sending messages. Let's see a few ways for that.

<h4>In text</h4>
This is a fun one. Not like the others, it needs some creativity.  Like when you read every starting letter of the sentences. Or this:


![original](/img/stego/forget.png)

<h4>In picture</h4>
Now let's get a more interesting one. How to put stuff into other pictures. Let's make it simple. I know that just using others program is not the best way, but for  now, we will just go that way.
We will use steghide this time. It can hide information in images and audio files. It is really good because you have to give a password to hide your message in the file. For example: You have an image and you want to hide it inside another image, you will need a password to retrieve the original image. You can't tell if there is a hidden file inside the image. How to use it? Steghide has no gui so you can only use it from command line. 
    steghide embed -cf mainImage.jpg -ef hiddenImage
then you need to enter the password. To retrieve the image:  
    steghide extract -sf mainImage.jpg
then you need to enter the password. As you can see, you cannot tell in advance if there is something hidden in the image. If there is no file or the password is incorrect, steghide will just fail. That is all. 

From here, Bob will post the file, Alice will download it, know the password and extract the image. 

Here are two files, one is the original one is with a hidden picture.(try party ;))
[from](https://photographylife.com/news/nikon-d810-high-resolution-image-samples)
![original](/img/stego/original.jpg)
![stego](/img/stego/stego.jpg)

Good:
Attacker can't see the messages
Attacker can't send messages in the name of someone else, if the messages list is unknown
Attacker will not know that a message has been sent. 

Bad:
Slow
Need a pre-shared password.

<h4>Finding steganography</h4>
To be honest, if someone uses a good stego method, it is really hard to find. Most of the times, the way stego users get caught, is if they have to original pictures with them. You can hash the two pictures and if the hashes are different, maybe you are dealing with a stego image. If you have access to the suspected stego users computer, and you find stego programs there, you can be sure they have some secrets.


<h2> Epilogue </h2>
Cryptography is essential in today's secure communication.The mathematics behind these algorithms, which was not part of this post, are fascinating. Steganography is in my opinion a fun way to hide things in plain site. Hope you enjoyed.


