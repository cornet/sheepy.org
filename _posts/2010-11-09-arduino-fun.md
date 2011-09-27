--- 
layout: post
title: Arduino fun
created: 1289342751
---
So after all the weekend plans got cancelled I decided to do a bit of tidying... right up until the point I discovered a female RJ45 socket on the bottom of my energy meter.

Doing what any self-respecting geek would do I plugged in a piece of cat5 and waved a DHCP lease at it. No dice, time for some research,

The energy meter in question is a re-branded [current cost classic](http://www.currentcost.com/product-theclassic.html) of which, at least on initial research, there are 2 versions. (I'll come back to this in a bit).

Now the basic principle as to how these devices work is thus: You clamp an induction meter round your main electric feed connected to a wireless transmitter which talks to a unit with an LCD which has this handy RJ45 socket. The RJ45 socket actually turns out to speak 3.3V TTL for which you can acquire a data cable from [here](http://stores.ebay.co.uk/Current-Cost-Ltd).

But it's Saturday and I want this working now! Enter, stage left, Arduino. Some reading of  [this](http://e.inste.in/2008/06/16/interfacing-the-currentcost-meter-to-an-arduino/) and [this](http://mungbean.org/blog/?p=477) there are 2 versions:
<ul>
<li>4200 Baud over pins 4 &amp; 8</li>
<li>9600 Baud over pins 6 &amp; 7</li>
</ul>

Mine obviously has some personality issues and decided that it would operate using pins 4 &amp; 8 at 9600 Baud.

A quick Ardunio sketch later I had it spewing out XML over the hardware serial port using [NewSoftSerial](http://arduiniana.org/libraries/newsoftserial/) to do the conversion. 

All going well I decided to punt this data out using the [Arduino Ethernet Shield](http://www.arduino.cc/en/Main/ArduinoEthernetShield).

Cue timing problems...

It appears the poor little Arduino couldn't cope and was missing/corrupting the output and was missing quite a lot of characters whitest trying to parse and punt data out over Ethernet. Throwing some buffers into the mix appears to have resolved the issue and it now appears to get 90%+ of the readings using this [sketch](https://gist.github.com/667488)

All in all a productive weekend and hopefully I should have some graphs soon to show what sort of power I'm consuming.

Current TODO list:
* Tidy up the code
* Write agent to connect to the Arduino and get current usage
* Stick this into my [munin](http://munin-monitoring.org/) install [here](https://secure.sheepy.org/munin)
