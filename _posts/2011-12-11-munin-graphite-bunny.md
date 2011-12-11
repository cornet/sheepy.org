--- 
layout: post
title: Munin and Graphite, using bunny this time.
---

After reading [this](http://www.devco.net/archives/2011/12/11/common-messaging-patterns-using-stomp-%E2%80%93-part-2.php) post by [@ripienaar](https://twitter.com/#!/ripienaar)
I decided it re-visit my munin-graphite setup to make it somewhat more elegant.

While the article advocates the use of stop I would prefer to stick with Graphite's built in AMQP support if possible but I agree that the amqp gem is far from elegant.
Not what you want if you're a sysadmin needing to solve a problem quickly.

I've seen [bunny](https://github.com/ruby-amqp/bunny/wiki) in passing but never used it before. Turns out it's quite nice. Also, again from a previous article from [@ripienaar](http://www.devco.net/archives/2011/10/02/interact-with-munin-node-from-ruby.php), the [munin-ruby](https://github.com/sosedoff/munin-ruby) gem caught my eye.

Not too long later I'd replaced my original implementation with something a lot simpler. As ever it needs some work whch I'll hopefully get round to this week however here is the new code:

<script src="https://gist.github.com/1463505.js?file=munin-graphite.rb"></script>
