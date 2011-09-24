--- 
layout: post
title: Sun Java on Debian/Ubuntu
created: 1221750563
---
It would appear that even if you install ONLY the sun java package it
drags in some GNU java stuff as well.

You can go through everything in /etc/alternatives and update it
manually but that is somewhat time consuming.

Instead just run:

<code>sudo /usr/sbin/update-java-alternatives --set java-1.5.0-sun</code>

and this will sort everything  :) 

