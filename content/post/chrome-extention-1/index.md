---
title: Using Chrome Extension To Track User Events (Part 1)
subtitle: Building a chrome extension to track user events on Stack Overflow
summary: Building a chrome extension to track user events on Stack Overflow
date: 2017-09-06
math: true
diagram: true
markup: mmark
tags:
- NodeJS
- Javascript
categories:
- Web
image:
  placement: 3
  caption: 'Image credit: [**Google Images**](https://google.com/images)'
---

<!-- <img src="/assets/images/school/AW/track.jpg" width="700"> -->

In this age of adaptation and personalization, user insight plays a key role. What is the user demographic? What is the user interested in? What type of content does user mostly consume? What's the surfing pattern of the user?
These are all sorts of questions that can reveal a lot about the audience. So how do we get answers to these questions?

It all starts with user tracking. 

Events like user comments, likes, up-votes, down-votes, scrolling pattern, time spent on the website can be tracked. This blog post discusses an approach to perform user tracking.

## Problem Statement:
>(a) Create a website that permits account creation and log-in.<br>
>(b) After the user is logged in, direct the user to a user profile page, which links to the page https://stackoverflow.com/questions/tagged/java?sort=frequent&pageSize=15<br>
>(c) Log the interactions upon the usage on the pages. (i.e. click, post, answer, comment, bookmark, thumb up/down, scroll to read, mouse movement trajectory, etc...)  

## First thoughts

There are various analytics libraries available such as [Google Analytics Library](https://developers.google.com/analytics/devguides/collection/analyticsjs/), [Hammer.js](https://hammerjs.github.io/), that can be simply plugged in and used.

However, there's a catch. Since navigating to Stackoverflow.com would take us out of our application's domain, I doubt if it's possible to track the user once he has navigated outside the domain of our web application. 

## Possible Approaches 

**1. Mirror site locally.**

One solution could be to mirror the stackoverflow.com website locally. Once the user clicks the link, the page can be downloaded and then rendered from the local server.
That would also give us the freedom to track all the user's events.

**Catch:**
Trying to mirror the whole website for the sake of tracking, is a bit of an overkill.

**2. Using a Browser Extension.**

Another solution could be installing a [Chrome extension](https://developer.chrome.com/extensions). The most apparent benefit is that it can let us track the user on the browser level, freeing us from any constraints of our web application's domain.

**Catch:**

This approach involves having the user to install the extension for us to track his events, which is a bit *interfering*, to say the least.

## Browser Extension Approach

### Technology Stack
<br>
**Extension type:** Google chrome extension<br>
**Server:** NodeJS<br>
**Client:** [EJS View Engine](http://www.embeddedjs.com/), HTML, Javascript, JQuery<br>
**Database:** MYSQL database.<br>

### Architecture

<img src="/assets/images/school/AW/extension_architecture.png">

### Strategy

Our strategy is to intercept user events using a browser extension, retrieve the information from the event and send the data to our server, which in turn send the data to the database server for storage.

### Components
<br>
**Chrome extension**

For building a chrome extension, we need 3 files:
1. manifest.json
2. inject.js
3. background.js


1 .**manifest.json**


    {
        "name": "AWAssignment",
        "version": "0.0.1",
        "manifest_version": 2,
        "description": "Tracking user events",
        "homepage_url": "http://thebotspeaks.com",
        "background": {
            "scripts": [
            "jquery-3.2.1.min.js",
            "background.js"
            ],
        "persistent": true
        },
        "browser_action": {
            "default_title": "Inject!"
        },
        "permissions": [
              "https://*/*",
              "http://*/*",
              "tabs",
              "http://localhost:*/",
              "http://192.168.0.15:*/"
            ],
        "content_scripts": [ {
              "js": [ "jquery-3.2.1.min.js" ],
              "matches": [ "http://*/*", "https://*/*"]
            }],
         "content_security_policy": "script-src 'self' https://cdn.socket.io; object-src 'self'"
        }


The key “background” is worth noticing here as it refers the second file required for building our extension: background.js.
```
“background”: { “scripts”: [ 
                    “jquery-3.2.1.min.js”, 
                    “background.js” 
                 ],
```

**2 . background.js**

```
// this is the background code…

chrome.tabs.onUpdated.addListener(function (tabId, changeInfo, tab) {
 // for the current tab, inject the “inject.js” file & execute it
 if (changeInfo.status == ‘complete’ && tab.active) {
 chrome.tabs.executeScript(tab.ib, {
 file: ‘inject.js’
 });
 }
});
```

This code controls when our extension should start working. For our project, we need to start tracking once the tab has loaded completely. 
This file refers to the final and core piece of our extension: Inject.js.


**3 . inject.js**

```
(function() {
//user tracking code goes here
})();
```


### Events to be tracked

Following user events can be tracked to get user insight.
1. Click information.
2. Mouse pointer coordinates information.
3. Ctrl-c information.
4. Ctrl-s information.
5. Scroll information.
6. Up-vote/Down-vote information.

To deploy the extension:<br>
Navigate to Chrome->More Tools->Extensions.<br>
Click on the ```load unpacked extension``` button and select the folder where your extension code resides.

**2 . NodeJS server**

We set up a NodeJS server to collect information from the browser extension and persisting the data.

**3 . Database (MYSQL)**

We use MYSQL database server for storing the user event data. Although I do see it getting changed to a NOSQL database such as MongoDB, owing to the type of data we are dealing here.

## NodeJS-Chrome Extension communication

Here’s a pain point. Majority of the internet has migrated towards [HTTPS from HTTP](https://www.instantssl.com/ssl-certificate-products/https.html). However, our NodeJS server uses HTTP protocol, since we don’t have a [SSL certificate](https://security.stackexchange.com/questions/16595/what-does-it-mean-for-a-digital-certificate-to-be-signed). Any attempt at communication between our NodeJS server and our browser extension would result in a [Mixed Content Error](https://developers.google.com/web/fundamentals/security/prevent-mixed-content/what-is-mixed-content).

So how do we solve this issue ?

We [create a self signed SSL certificate](https://msol.io/blog/tech/create-a-self-signed-ssl-certificate-with-openssl/) to calm down our browsers who are still red from freaking and shrieking about that “HTTP” in the address bar.

**Important thing to remember**

```
openssl req -new -sha256 -key key.pem -out csr.csr
```

This command generates a [Certificate Signing Request](https://www.wikiwand.com/en/Certificate_signing_request), which you could instead use to generate a CA-signed certificate. This step will ask you questions; for the key “Common Name (e.g. server FQDN or YOUR name)”, enter the value ```localhost```.

Once you have created the SSL certificate, you will have two files:<br>
1. key.pem
2. certificate.pem

You will need these files to set up [HTTPS nodeJS Server](https://stackoverflow.com/questions/5998694/how-to-create-an-https-server-in-node-js/14272874#14272874).

Now that you have set up the server and the extension, time to test the solution.

Enter ```https://localhost:443``` in the browser address bar.

You will see a screen like this.

<img src="/assets/images/school/AW/error.jpeg">


Click Advanced->Proceed. And you are good to go.