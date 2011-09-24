--- 
layout: post
title: Bash Cures Cancer
created: 1203896867
---
Another site that I've just added to my feed list.

Quite a few things on here I wasn't aware of such as:
<code>
find . -name 'file-*' -delete
</code>
is much faster than
<code>
find . -name 'file-*' -exec rm {} \;
</code>
although I need to test how it compares to:
<code>
find . -name 'file-*' | xargs rm
</code>

<a href="http://bashcurescancer.com/">BASH Cures Cancer</a>
