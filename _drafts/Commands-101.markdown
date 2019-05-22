---
layout: post
title:  "Basic Commands"
date:   2018-05-29 18:05:55 +0300
image:  OSINT1.PNG
tags:   [101, OSCP]
---
You probably know most of the commands. Maybe even thinking about leaving this page.Like when you skip the first few page of a tutorial because you already know that. Maybe you do, but let me try to show you some interesting stuff.

First, lets just sum up the basic commands:

| Command              | Linux | MS-DOS | PowerShell    |
|----------------------|-------|--------|---------------|
| Copy file            | cp    | copy   | Copy-Item     |
| Move file            | mv    | move   | Move-Item     |
| List files           | ls    | dir    | Get-ChildItem |
| Clear screen         | cls   | clear  | Clear-Host    |
| Display File Content | cat   | type   | Get-Content   |

So that is about the basic. Let's look at Linux a little bit for now. How Linux works is that if you write a command, then it will go and try to find a file with the same name. It checks the Path variable folder by folder and execute the first file with the same name. So instead of cat we can write /bin/cat.

![Full Path Cat](/img/full-path-cat.png)

What if we don't write down the full path. We can use wildcards. ? will be 1 char * will be any number. So what happens if we write /bin/ca? 

![Wildcards](/img/wildcards.png)
