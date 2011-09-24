--- 
layout: post
title: Nokia MMSC Server
created: 1217014159
---
Nokia's MMSC is a stunning example of how to make problems as hard as possible to debug.

The MMSC can speak a number of protocols over HTTP including EAIF and MM7. As you would expect you submit your POST and get back a HTTP Error Code and Message. However the Codes and Messages leave a lot to be desired:

So lets take the 499 Error code which, according to their spec, can mean:
<ul>
<li>Address Hiding - basically reject all messages to this Number</li>
<li>Desired Delivery Time Error - aka delivery time set too far in the future</li>
<li>No Credit</li>
<li>Recipient Barred</li>
<li>Sender Barred</li>
<li>Rejected - or "We ain't gonna tell you why"</li>
</ul>

No surprises for guessing which one I'm seeing at the moment :)

Next comes the legendary 599 Error Code, again lets look at the spec.
This can mean:
<ul>
<li>CCR Error - Rejected as the MMSC has exceeded the number of messages licensed</li>
<li>Capacity Exceeded - Number of msgs/second exceeded</li>
<li>Kernel Overloaded</li>
<li>DB Error - Message could not be stored in database</li>
</ul>
although in our case we normally see "No reason given or reason given not included in EAIF specification". 

If the above wasn't fun enough, Nokia obviously doesn't think it is, if you use the Nokia MMSC Client libs they will generate their own 666 error code if they don't get anything back from the MMSC. Great!

Seriously, how do people manage to come up with this crap. If you're going to define Error codes and Messages then FFS use them and don't have a <em>"Oh we can't be bothered to tell you why"</em> message.

