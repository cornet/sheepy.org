--- 
layout: post
title: Railscasts
created: 1207361726
---
So I was pointed in the direction of <a href="http://railscasts.com/">Railscasts.com</a> and duly leeched the site:

<code>
curl "http://feeds.feedburner.com/railscasts_ipod"  \
| grep -Eo "http://media.railscasts.com/ipod_videos/.*m4v" \
| sort | xargs -n1 wget -c
</code>

did the trick.

Now I need to find time to watch them all :)
