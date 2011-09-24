--- 
layout: post
title: Hard disk monitoring
created: 1147911903
---
IDE drives are basically crap. However they are cheap so most people use them.

I’ve recently discovered smartmontools which monitors most hard disks using S.M.A.R.T which allows the disk to do self testing.

Setting up on gentoo is fairly easy and should hopefully give me some advanced warning of disk failure. Together with mdadm , which monitors raid arrays, then you can minimise the risk of loosing data.

**Big fat hairy warning**
This is NOT any reason not to back up your data.

On with the install and config.
<pre>
emerge -v smartmontools
</pre>
Now create /etc/smartd.conf and have it contain something like the
following:
<pre>
/dev/hda -a -m nathan@domain -s (S/../.././02|L/../../4/05)
/dev/hdb -a -m nathan@domain -s (S/../.././03|L/../../5/05)
/dev/hdc -a -m nathan@domain -s (S/../.././04|L/../../6/05)
</pre>
The usual start on bootup.
<pre>
rc-update add smartd default
/etc/init.d/smartd start
</pre>

This config checks all supported attributes of the disks performs a short self test once a day at 2am, 3am and 4am for hda,b and c respectivly. It also does a long self test once a week at 5am on Thurs, Fri and Sat, again respectivly.
If a test produces an error then it emails nathan@domain telling me so.

Also check the man page for smartctl for more options and details.

Next up - mdadm
This is fairly trivial to set up and get working. Again it emails you when there is a problem.

Install
emerge -v mdadm

Create the mdadm.conf with details of our RAID array
<pre>
mdadm --detail --scan > /etc/mdadm.conf
</pre>

Set your email address
<pre>
echo "MAILADDR nathan@domain" >> /etc/mdadm.conf
</pre>
Make the monitor start on boot, and start it now.
<pre>
echo "mdadm --monitor --scan --daemonise --config=/etc/mdadm.conf" >> /etc/conf.d/local.start
/etc/init.d/local restart
</pre>
