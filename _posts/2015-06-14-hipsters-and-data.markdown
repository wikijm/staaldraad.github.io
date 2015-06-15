---
layout: post
title:  "Hipsters and data"
date:   2015-06-14 18:23:39
categories: databases security nosql
---
A while back I spent some time playing with the “modern” database implementations, more affectionately known as hipster tech. These are mostly your so-called NoSQL, big-data, ect databases. Trying to interact with these databases required numerous scripts to be written, one for each database implementation.  After chatting to [@PaulWebSec][paultw] I decided to merge these into a single tool. Thus [HippyDB Tool][hippydb] was born.

The following databases are supported:
* Aerospike
* Cassandra
* Hbase
* Hive
* Memcached
* Mongodb
* Redis
* Riak

A quick scan of AWS and Google Cloud hosting showed that the vast majority of these databases are deployed on default ports, listening on all interfaces and most critically, without any authentication. Furthermore, [Shodan reports][shodan] around ~59k MongoDB instances on the default port of 27017 and again, all the data is available for all to view.  

Having a tool to easily interact with these deployments hightlights just how much data is available and what damage this could cause. Numerous instances of user creds, personal information and even credit-card numbers can be found. 

The tool is written in NodeJS as it just felt write to write in a "hipster" language.. 
To use HippyDB, simply fork the [Github project][hippydb], install the requirements and you should be good to go.

{% highlight bash %}
$ git clone git@github.com:staaldraad/hippydb.git
$ cd hippydb
$ npm install
$ nodejs hippydb.js 
{% endhighlight %}

Interaction is pretty straight forward, with a menu driven interface. `help` will give you all the available commands and the tool even includes tab-completion! A sample session against a Riak instances may look as follows:
{% highlight bash %}
$ nodejs hippydb.js
[*] Hippy Database interaction tool.
[*] Author: etienne@sensepost.com

[*] Type help to get started
> set host 10.10.10.1
[*] host=10.10.10.1
> riak help
[*] Riak dumper -- Help
Commands: 
> riak status                       //Check if we can connect to the database
> riak buckets                      //list all buckets on the host
> riak keys  <bucket>               //list all keys in a bucket
> riak dumpk <bucket> <key>         //dump value of key
> riak dump  <bucket>               //show 'limit' number of values in a bucket

> riak status
[*] Connected! Node is [riak@bbdev1.10.10.10.1]
> riak buckets
[*] Found [10] Buckets
private_storage
privacy
offline_msg
passwd
roster
...<snip>...

> riak keys passwd
[*] Found [70] keys
x123545@10.10.10.1
...<snip>...

> exit
Cheerio old-chap
{% endhighlight %}

Happy hipster hunting and please fork and contribute!

[paultw]: https://twitter.com/paulwebsec
[hippydb]:      https://github.com/staaldraad/hippydb
[shodan]:   https://www.shodan.io/search?query=port%3A27017
[jekyll-help]: https://github.com/jekyll/jekyll-help
