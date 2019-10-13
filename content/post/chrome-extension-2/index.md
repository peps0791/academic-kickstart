---
title: Using Chrome Extension To Track User Events (Part 2)
subtitle: Building a chrome extension to track user events on Stack Overflow
summary: Building a chrome extension to track user events on Stack Overflow
date: 2017-09-10
math: true
diagram: true
markup: mmark
tags:
- NodeJS
- Javascript
- MongoDB
categories:
- Web
image:
  placement: 3
  caption: 'Image credit: [**Google Images**](https://google.com/images)'
---

<!-- <img src="/assets/images/school/AW/track.jpg" width="700"> -->

*Disclaimer: This post is in continuation to the [last post](http://thebotspeaks.com/2017-09-06-Using-Chrome-Extension-To-Track-User-Events-Part-1) I wrote about setting up a Chrome extension-NodeJS framework to track user events on Stackoverflow.com.*

Though the project setup described in the last post solves the purpose of tracking user events, there were a few limitations with the approach such as:
1. NodeJS server wasn't hosted, which meant setting up of framework manually to test the functionality.
2. Generation of a self-signed certificate wasn't that great either in terms of convenience.
3. Use of MYSQL as database definitely wasn't the best choice as querying of data required applying joins on multiple tables.
4. The user data was being sent to the server in real time, which isn't the ideal choice in terms of using network bandwidth.

So, having identified the problems, our new goals are: 
1. Moving the whole application (the NodeJS server and the database) online.
2. Finding an alternative to self-signed SSL certificate to enable Chrome extension-NodeJS communication.
3. Migrating from MYSQL to a NoSQL database.
4. Tweaking the data sending strategy to put less load on the network and database server.

## Hosting the NodeJS server using the Heroku platform

>[Heroku](https://dashboard.heroku.com/) is a cloud platform for hosting web applications.

1 . Sign up to create an account.<br>
2 . Create a git repository for your NodeJS project.<br>
3 . Create a new file named ```Procfile```.<br>

Add following content to the file.

```
web: node index.js
```
where index.js is the starting file for your NodeJS server.

4 . Push the entire code in the git repository.<br>
5 . From within the project folder, execute the following command from the command line:<br>

```
heroku login
```

6 . Enter your Heroku credentials.<br>
7 . Next, run the following command:<br>
```
heroku create <app name>
```

The output should look like this:
```
Creating ⬢ <app name>... done
 ▸    ENOENT: spawn git ENOENT
```

8 . Go to [your Heroku dashboard](https://dashboard.heroku.com/)<br>
9 . You will see your app that you created via command line.<br>
10 . Click on the app.<br>
11 . You will see various tabs like Overview, Resources, Deploy, Metrics etc.
Go to the ```Deploy``` tab.<br>
12 . In the ```Deployment``` section, select the Github option and connect your Github account with Heroku<br>
13 . In the ```App connected to GitHub``` section, select your NodeJS repository and connect it with the Heroku application.<br>
14 . Finally, deploy the code directly from Github repository by clicking on the ```Deploy Branch``` button.<br>
15 . You will see the deployment logs. Once it's successful, click on ```open app``` button on the top of the page and you can now access your NodeJS server using the URL that you see in the address bar of the browser.


**Pro Tip**: *If you specified the hostname and port information for your NodeJS server something like this:*

```
httpServer.listen({
  hostname: 'localhost',
  port: 3000
}, function(err, result){
  if(err)console.log('Some error starting the server');
  console.log('App server started successfully...')
}
```


Heroku will have serious problems with this, as it won't accept you telling it which port or hostname to run on.

To work around this problem:
1. Remove the ```hostname``` key-value pair.
2. Re-write the ```port``` key-value pair in the following way:

```
port: process.env.PORT || 3000
```

Which is just letting Heroku use the port of its own choice for running your application.

## What about the SSL requirement for Chrome extension-NodeJS Communication?

We make use of the [SSL termination](https://f5.com/glossary/ssl-termination) feature provided by Heroku here. If you noticed, the URL of our hosted application is an HTTPS one, and hence, the communication between our extension and the Node application is encrypted. At the Heroku endpoint though, the encryption of data ends and the traffic between the Heroku platform endpoint and our NodeJS server is unencrypted. But we can let it slide for now and can do away with having to create a self-signed certificate.

### What about when testing with the local server?

Glad you asked!
[ngrok](https://ngrok.com/) is the answer here. 
ngrok enables exposing the local server to the outside world by providing an access URL. Which is great!
And it doesn't take much to set up either. Just download the ngrok binary.

Now once you start your local NodeJS server, 
1. Open another command line prompt and run the command
```
ngrok http <your NodeJS server port>
```

You will notice that it spits out both HTTP and HTTPS URLs.
We need the HTTPS URL to let our extension speak with the server.
Use this URL as the server address in the extension to access the NodeJS server.

**Pro Tip:** 
*ngrok creates a new URL each time the server is fired, so you will have to change the URL each time in your extension.*

## Database Migration from MYSQL to NoSQL

We make use of [MongoDB](https://www.mongodb.com/), a document based database as our NoSQL database. We also make use of the [mongoose](http://mongoosejs.com/), NodeJS library to integrate MongoDB with NodeJS.
However, before moving onto integration, we need an online MongoDB database to keep in line with our primary goal of moving the whole framework online.
Fortunately, we have that sort of thing available in [mlab.com](https://mlab.com).<br>

The free account includes 500MB of storage, which is more than enough for our application.

Create an account on mlab.com, then proceed to create a collection.
Once you have created the collection, create a new database user for this collection.

**Integrating MongoDB with NodeJS**

    connect: function(callback){
        mongoose.connect(```dbUri``);
        var db = mongoose.connection;
      db.on('error', function(err){
          //conole.log('DB connection error' + err);
          callback(err, null);
      });
      db.once('open', function(){
          callback(null, 1);
      });
},

For dbUri, use- 
```
mongodb://<dbuser>:<dbpassword>@ds133814.mlab.com:33814/<db_name>
```


Here, ```dbuser``` is the user you created for your collection on mlab.com.<br>
And ```db_name``` id the name of the collection you created.

You can also find the URL details on mlab.com for your created collection.


## Improved way of logging:

Up til now, we were sending the data to the server (and database) in real time as we were receiving the user event information on the client side.
This led to considerable load on the bandwidth as packets were being sent consecutively, which led to slowing down of the database as the incoming requests kept on stacking up fast while the database was dealing with the previously sent requests.

To get around this issue, we will make use of browser storage to store the data for a certain period of time and send the data to the server in batches, essentially.

For storage, the global variables of javascript come to mind, but there's a catch. Javascript files are reloaded with every new URL visit and refresh. And with each reload, the variables are re-initialized. which would render our strategy helpless. 
But fortunately for us, we can use [browser localStorage](https://www.w3schools.com/html/html5_webstorage.asp) to persist data across page reloads and re-directs.

And accessing local storage is not that complicated either.
 
```
localStorage.setItem('events', JSON.stringify(events))
```


```
localStorage.getItem('isTimerSet')
```

**Pro Tip**: 
*Objects cannot be set as values to keys in local storage. It only accepts String as keys and values. So if you try saving a JSON object, it will get saved as "[Object]". which is not what we want.*<br> 
*Work Around: Use JSON.stringify(Obj) before saving to local storage.*

## Hosting the Chrome Extension:

Here's the part where you will have to pay to get everything online. Google Chrome requires you to create a [developer account](https://chrome.google.com/webstore/developer) and pay $5 as verification fee for hosting your extension on the Chrome Web store. After creating the acocunt, and paying the fee, you can host the app by just uploading the zip folder of your code.

And that's it! We are done with the first part of this project. We tracked user events, stored them in the database for later analysis and all of this is hosted online.




