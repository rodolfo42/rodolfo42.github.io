---
layout: post
title: Real time event notifications for web apps - Part 1
date: 2013-02-14
comments: true
---

## Overview

I thought I would share an interesting solution I came up with for notifying active users of a web application about system events in real time.
I'll just use the following scenario:

> an e-commerce app which needs to notify **active** back-office administrators in the instant when an order has been placed or when an order's items have just been successfully delivered to their destination.

Note the emphasis on **active** - what I mean is: currently logged in and active users, with at least one tab open in any page of the app. An on-screen, desktop notification is appropriate, and with Chromium's [Desktop Notifications API] [DesktopNotifications] it's not only possible to notify the user when the tab is active, but even when it's not (and even when the whole browser itself is minimized).

Of course you could have e-mail alerts, but that could end up cluttering every inbox with unnecessary and outdated alerts, once the order could have already been acted upon long before these alerts would be checked.

<!--more-->

## Architecture

So in order to make it *actually* real time, the idea is to have an *event server* which is responsible for only two things:

- keeping a list of active listeners for events (subscribers)
- react upon receiving event data from the app (publisher), by pushing it to all active listeners, using a common data interchange format

Basically, an implementation of the old [Publish-Subscribe model] [PubSub]. The app doesn't care if there are any subscribers for the event, and the subscribers don't care if there are any publishers. To illustrate:

![Diagram describing the relations between the app, event server and the browser clients] [Diagram1]

Pretty straightforward.

The communication between the app and the server has an important attribute: the server doesn't wait until it has finished notifying all the subscribers, in order to send a response back to the app. It's **asynchronous**, meaning you could have 1000's of clients connected, and still the event server would respond quickly to the app. Non-blocking communication like that it's what makes it able to handle high ammounts of traffic.

The event server's response to the app would only indicate that the server received the event data successfully. Whether it has tons of clients to notify or none, the app will still get the same response time, allowing it to scale well. I have found that this approach also decreases coupling and thus becomes much more modular and mantainable.

In the [second part]({% post_url 2013-02-20-real-time-event-notifications-for-web-apps-part-2 %}) of this post, I will share the actual code I wrote for this solution and the technologies and languages used.

[DesktopNotifications]: http://www.chromium.org/developers/design-documents/desktop-notifications/api-specification "Chromium Desktop Notification Spec"
[PubSub]: http://c2.com/cgi/wiki?PublishSubscribeModel "C2 Wiki on Pub/Sub Model"
[Diagram1]: /uploads/arch1.png
