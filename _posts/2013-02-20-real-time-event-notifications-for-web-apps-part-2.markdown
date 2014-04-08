---
layout: post
title: Real time event notifications for web apps - Part 2
date: 2013-02-20
comments: true
---

## Overview

So, in my [previous post]({% post_url 2013-02-14-real-time-event-notifications-for-web-apps-part-1 %}), I introduced the idea of a solution for realtime event messaging using a Pub/Sub implementation and Chromium's Desktop Notifications API.

The architectural view presented there only covers the big picture. Now let's get into details technically.

In summary, the solution presented here will provide a organized way of having a event server available for publishing events to active users. This is a web-based browser-only solution, as it uses WebSockets and the Socket.IO library.

> **Note:** The notification pop-ups are implemented using Desktop Notifications in order to have the notifications appear outside the tab, even when the browser is minimized. This only works in Chrome and Safari right now. If you really need this outside Chrome, you can use an [extension for Firefox](https://addons.mozilla.org/en-us/firefox/addon/html-notifications/), but other browsers don't have a similar solution as of today.

<!-- more -->

## The server

In order for the communication between the event server, app and it's subscribers to be **asynchronous** as mentioned, I chose to use [Node.js] [NodeJS] from Joyent, due to it's non-blocking, evented I/O model. It leverages system's resources more efficiently when holding many concurrent connections at once, and it allows us to do concurrent network programming in a very straightforward manner. It works for me, so it's really _not_ because it's fancy or cool.

We will also use the [Express] [Express] framework, since it simplifies the task of routing, parsing requests and sending responses.

## The client and the communication

As we're talking Javascript, the data interchange format used by this particular case will be JSON.

And for the core "realtime behaviour" of this scheme, we'll want to keep a permanent connection between the event server and it's subscribers. As they're nothing but browser clients, the choice is made towards [Socket.IO] [SocketIO], a realtime library that provides various means to enjoy Comet-style connection between the browser and the remote server, listening for and emitting (pushing) events.

## The code

We'll have javascript code on both server and client sides.

### Server-side

We begin by coding an `app.js` in a clean working directory:

{% highlight js %}
var application_root = __dirname,
    path = require("path"),
    fs = require("fs"),
    http = require('http');
{% endhighlight %}

Now, we will configure the express framework, for exposing our API for publishing events.

{% highlight js %}
// port the server will listen on
var listenPort = 3000;

var express = require("express");
var app = express();

var server = http.createServer(app);
var io = require('socket.io').listen(server, {log: false});

app.configure(function () {
    app.use(express.bodyParser());
    app.use(express.methodOverride());
    app.use(express.errorHandler({ dumpExceptions:true, showStack:true }));
    app.use(express.static(__dirname + '/public'));
});
{% endhighlight %}

We will use [log4js] [Log4JS] to get a cleaner log output for debugging and monitoring purposes.

{% highlight js %}
var log4js = require('log4js')

log4js.configure({
    appenders: [
        {type: "console"}
    ],
    replaceConsole: true
});

var log = log4js.getLogger();
{% endhighlight %}

Declare your dependencies and project info in a `package.json` file in the root:

{% highlight js %}
{
    "name":"event-server",
    "version":"0.0.1",
    "private":true,
    "scripts":{
        "start":"node app"
    },
    "dependencies":{
        "express":"3.1.0",
        "socket.io": "0.9.13",
        "log4js": "0.5.6"
    }
}
{% endhighlight %}

After this setup, we can begin defining our API endpoints. We will first create a POST route that receives event data and publishes it. This can be anything you want, but for now let's assume we only need a `title`, `message` and `url` for the notification to be displayed. The data will be received by our server in a JSON string in the HTTP request body.

> **Note:** Express [also supports] [ExpressBodyParser] urlencoded and multipart for parsing the request body, when using `bodyParser()` as above, but we will focus on JSON for brevity

Let's create this route as `/events`:

{% highlight js %}
app.post('/events', function (req, res) {
    var data = req.body;
    log.info('POST /events');

    // verifies the data is indeed valid
    // since Express automatically parses JSON body, it should be an object and non-empty
    if(typeof data == 'object' && data != {}) {
        log.debug(JSON.stringify(data));
        res.send({status: "success"}); // note the asynchronicity
        log.debug('returned success status');

        // notify clients after we are done responding
        setTimeout(function(){
            log.debug('pushing updates to subscribers..');
            broadcastEvent('newMessage', data);
        }, 1);
    } else { // express couldn't parse the body
        log.error('error parsing the request body');
        log.error(data);
    }
});
{% endhighlight %}

Notice that are not defining/validating any specific structure for the event data, but you can easily do that here, and just return a different status in case of an error.

The `broadcastEvent` function is self-explaining - it's responsible for broadcasting the event to all registered listeners of the event server. So let's look at it's code:

{% highlight js %}
var onlineSockets = new SocketMap();

// emit events to the top sockets of all slots, excluding the `senderSocket`
var broadcastEvent = function(eventName, data, senderSocket) {
    senderSocket = senderSocket || false;
    var allSockets = onlineSockets.getList(), currSocket, qtyNotified = 0;
    onlineSockets.debug();
    for(var i=0; i < allSockets.length; i++) {
        currSocket = allSockets[i];
        if(senderSocket && senderSocket == currSocket) {
            continue;
        }
        if(currSocket != null) {
            qtyNotified++;
            currSocket.emit(eventName, data);
        }
    }
    log.info(qtyNotified + ' sockets where notified');
};
{% endhighlight %}

Now, we have a `onlineSockets` variable, which is an instance of `SocketMap`. This is a data structure created to hold all active sockets, but in a very specific manner, in order to solve the problem of having multiple tabs open on a page of your your app. The basic scheme is illustrated below;

![Diagram describing the SocketMap scheme] [Diagram2]

The squares A, B and C demonstrate the open tabs in the browser, each of those having one open WebSocket listening to events to be published by the event server. Each socket, as soon as it's connected, sends a `register` event passing a *hash* which is unique across all open browsers/sessions, but **shared between tabs** (this can be, for instance, your session ID). The reference to the socket then gets stored in the `SocketMap`, which creates "stacks" keyed by each new hash it receives.

So, when the socket registers with an existing hash:

- the new socket gets pushed to the stack of open sockets for that hash
- it becomes the one active socket for that hash

As soon as a tab is closed, it's socket loses connection to the server, and is automatically removed from the stack, whichever position it was in. If it was the last one (the active one), it's removed and the socket below it becomes the active socket for that hash.


### Client-side

In the client side, all we need to do is to insert the same script in all desired pages of the app. This script must:

- be able to access/generate a unique hash, shared between the tabs (cookie value for the session ID)
- register an socket for that tab, using said hash
- receive notifications and notify the user using Desktop Notifications API

So in order to get the hash, you can use a js library like [jQuery.cookie][jQueryCookie] to get the cookie that represents your session ID. In this particular case, I'll just generate a random hash and register it as a cookie, to simulate a real app's session ID:

{% highlight js %}
// your app cookie key
// eg. PHPSESSID or JSESSIONID
var COOKIE_KEY = "MYCOOKIEKEY";

// just for test purposes, create a cookie with an random number
// inspired by http://stackoverflow.com/a/2117523
if(typeof hash == 'undefined') {
    hash = 'xxxxxxxx'.replace(/[xy]/g, function(c) {
        var v = Math.random()*16|0;
        return v.toString(16);
    });
    $.cookie(COOKIE_KEY, hash, {expires : 3600});
}
{% endhighlight %}

To register the socket with the hash on the server, as we're using Socket.IO, we need to bind a callback to the `connect` event, which is fired right after the socket connects to the server:

{% highlight js %}
var webSocket = io.connect('http://event-server.example.com/');

webSocket.on('connect', function() {
    var data = {"hash": hash};
    webSocket.emit('register', data);
});
{% endhighlight %}

And finally, use the [Desktop Notifications API][DesktopNotifications] (Chrome/Safari only, FF via extension) to notify the user of events published by the event server:

{% highlight js %}
webSocket.on('newMessage', function(payload) {
    // sanitize event attributes
    var title = payload.title;
    var message = payload.message;

    // creates a DesktopNotification without the icon
    var n = (webkitNotifications.createNotification("", title, message));

    // when clicked, the notification should close and the user redirected to the URL, if provided
    n.onclick = function(e) {
        if(payload.url) {
            // open url in new tab
            window.open(payload.url, '_blank');
        } else {
            // otherwise, bring the tab into focus
            window.focus();
        }
        // closes the notification popup
        this.close();
    };

    n.show(); // important - display the notification!
});
{% endhighlight %}

In the code above, I added support for a `url` attribute, which, if present the incoming event data, loads the specified url when the notification is clicked.

## Bottomline

What's briliant about this solution, is that, in order for the user not to be harassed by a lot of notifications for each open tab, which can be 10, 20 or a 100 tabs, the server only publishes events to the *active* sockets (represented by the continuous black line), which are basically the last sockets to register for each hash.

---

### Demo

A demo is available on-line [here][DemoLink].

Just enable Desktop Notifications via the switch (accept the request for permission), open multiple tabs and fire a message via the web interface. You'll only receive one notification. As you close the tabs and fire messages, you will continue to receive only one notification, though all open tabs are listening.

You can invite some of your friends to open this same URL in their browsers and start a remote chat!

---

### Source code

The source code for this entire solution is available in the Github repository below.

<div class="github-widget" data-repo="rodolfo42/event-notifications"></div>

[NodeJS]: http://nodejs.org/ "Joyent's Node.js official website"
[Express]: http://expressjs.com/ "Express framework website"
[SocketIO]: http://socket.io/ "Socket.IO website"
[Log4JS]: https://github.com/nomiddlename/log4js-node "log4js on github"
[ExpressBodyParser]: http://expressjs.com/api.html#bodyParser "bodyParser() API reference"
[Diagram2]: /uploads/arch2.png
[jQueryCookie]: https://github.com/carhartl/jquery-cookie
[DesktopNotifications]: http://www.chromium.org/developers/design-documents/desktop-notifications/api-specification "Chromium Desktop Notification Spec"
[DemoLink]: http://evntsrvr.aws.af.cm/
