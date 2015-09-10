---
layout: post
title:  "Viewing, modifying and replaying websockets"
date:   2015-09-10 14:16:39
categories: security webapps html5 websockets
---

A few assessments back I ran into a web app that made libral use of websockets. Now it's been a while since encountering an app that makes use of websockets for such a large portion of it's content and I'd forgotten what a PITA it can be trying to intercept and replay websocket requests.

Seeing as it was a web app assessment I naturally turned to trusty (Burp Suite Pro )[https://portswigger.net/burp/]. Burp has supported websockets for a long while now, but far as I could find, this support only allowed you to intercept websocket requests and view/modify them. There was no option to set intercept rules based on websocket content, and also no options for replaying a specific websocket frame or using a frame with Intruder.

To help fill the gap of these missing functions I quickly slapped together a websocket proxy that can be used as an upstream proxy by Burp. This means we could use Burp for the awesome interception and standard HTTP tricks it can do, and we use the websocket proxy to do the 'repeater' and 'intruder' component. 

I've written a full post about this over at the SensePost blog (https://www.sensepost.com/blog/2015/another-intercepting-proxy/)[https://www.sensepost.com/blog/2015/another-intercepting-proxy/]. So if you are interested in some more of the technical details / screenshots, head over there.

The proxy is available on github at: (github.com/staaldraad/wsproxy)[https://github.com/staaldraad/wsproxy]. 
Please feel free to let me know about any bugs and updates are very welcome. Hopefully this is useful to someone else down the line. 

A quick note about websockets and the 'repeater', yes it isn't a true repeater as it creates a new websocket channel, however, since websockets have no idea of authentication and tend to use standard HTTP tricks for sessions, it allows us to do a fairly decent job. 
Websockets can also consist of binary data, currently the proxy doesn't allow you to view or repeat this type of traffic. Text data (usually just JSON) will work fine.

