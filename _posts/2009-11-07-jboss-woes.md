--- 
layout: post
title: JBoss woes
created: 1257559786
---
JBoss, on the whole, does hold up surprisingly well. This can probably be attributed to our skilled developers who, to be fair, I don't always give enough credit to.

However every so often JBoss plays up and the symptoms presented seem to point to obvious problems. However all is not as it appears.

It's release day and shiny new software is just itching to be deployed. The sysadmin gets up early and rocks up to the office to go through the standard deployment procedure. If only things always went to plan!

We have a cluster of a number of servers and use the <a href="http://www.jboss.org/community/wiki/JBossFarmDeployment">JBoss Farm Deployment</a> service to deploy applications. It's fairly straight forward, you build and deploy .ear, .war, .spring, etc... files to the <em>$JBOSS_HOME/server/default/farm/</em> directory and all the nodes pick up the new code.

Here comes the first gotcha, which we have known about for quite a while now. If you re-farm an already running package then by default it won't free itself from the PermGen heap so continuous redeploying will eventually mean you run out of PermGen memory.

The solution we have is to make sure we restart every node after deployment to clear out this memory.

What I've found out recently is this issue can be resolved by setting the following Java options:
<code>
-XX:+CMSPermGenSweepingEnabled -XX:+CMSClassUnloadingEnabled
</code>
which will free up the PermGen memory.

This will be going into testing out or dev and test environments shortly and hopefully on live (once we are sure there are no adverse affects). This should mean no restarting required in most cases.


I say in most cases as we do have some applications we can't "hot deploy". We have been instructed to do the following:

<ul>
<li>Shutdown the build node</li>
<li>Build and farm the application</li>
<li>Shutdown all other nodes</li>
<li>Bring up the build node</li>
<li>Bring up the other nodes</li>
</ul>

This obviously leads to a complete outage lasting a few minutes, but we can live with that for the most part.

Once such morning a co-worker followed this procedure and all appeared to go fine. However not long after some of our application started throwing "Broken Pipe" exceptions. The applications in question were communicating with our JBoss clusting using RMI. From the exceptions this initially looked like some network issue. The load balancers (LVS ones) were checked but no issues. More investigation required...

The nodes throwing the exception were part of a 6 node Tomcat cluster communicating to a 3 node JBoss cluster. On closer exception only nodes 1 and 4 of the Tomcat cluster were throwing exceptions. These were restarted but to no avail.

Then I remembered that we we do Source Hashing on our LVS nodes. Source hashing is used to make sure the same clients hit the servers, normally for session tracking purposes, this helped with diagnosis.

I found that 1 JBoss cluster node was at fault, but there were no exceptions in the logs. Further more most transactions were working fine. Just to be safe JBoss was restarted on the offending node but no difference. On with investigation I guess...

Eventually I found something that didn't make sense. <em>$JBOSS_HOME/server/default/tmp/deploy</em> had a timestamp older than I expected.

This directory is used to hold the expanded files from <em>$JBOSS_HOME/server/default/farm/</em> and <b>should</b> disappear when JBoss is shut down. I shutdown JBoss again and, for whatever reason, it still remained. So I deleted the directory by hand and started up JBoss. Sure enough the "Broken Pipe" exceptions disappeared.

I shutdown JBoss on offending node again and this time it removed the directory. Started up again and all fine.


After much playing around I've no idea what causes this. I know that if JBoss doesn't shut down correctly then this directory can remain causing clustering issues (which really don't make sense to me) but I've seen a number of occasions on our Test Environment where this directory has remained after a successful shutdown.

To make sure this doesn't happen again I've modified out start scripts to check for the presence of this directory and refuse to start up if it exists.

Fingers crossed we won't see this issue again.

