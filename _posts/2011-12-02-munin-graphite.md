--- 
layout: post
title: Munin and Graphite
---

[Munin](http://munin-monitoring.org/) is a great piece of software for monitoring servers. However it's well known that it's central poller sucks at scale.

[Graphite](http://graphite.wikidot.com/) looks like just the ticket for collating stats so I spent of my spare time recently investigating. After some research I found [munin-graphite](https://github.com/adamhjk/munin-graphite) which was what I wanted to punt data from Munin to Graphite. However I wanted to decouple the two by using AMQP.

Graphite now supports AMQP nateively although doesn't admit to it. If you check out the latest version you'll find carbon.amqp.conf.example which contains all you need to get started.

So now I've got Graphite consuming from AMQP I just needed to modify [munin-graphite](https://github.com/adamhjk/munin-graphite) submit it's data how I wanted.

The result of which is my own [fork](https://github.com/cornet/munin-graphite). Just customise munin-graphite-daemon.rb to your needs. I've hhad it running for around a month now with no problems. YMMV as always.


