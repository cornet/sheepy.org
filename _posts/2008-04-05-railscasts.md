--- 
layout: post
title: Railscasts
created: 1207361726
---
So I was pointed in the direction of [Railscasts.com](http://railscasts.com/) and duly leeched the site:

    curl "http://feeds.feedburner.com/railscasts_ipod"  \
       | grep -Eo "http://media.railscasts.com/ipod_videos/.*m4v" \
       | sort | xargs -n1 wget -c

did the trick.

Now I need to find time to watch them all :)
