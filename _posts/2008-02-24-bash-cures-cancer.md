--- 
layout: post
title: Bash Cures Cancer
created: 1203896867
---
Another site that I've just added to my feed list.

Quite a few things on here I wasn't aware of such as:

    find . -name 'file-*' -delete

is much faster than

    find . -name 'file-*' -exec rm {} \;

although I need to test how it compares to:

    find . -name 'file-*' | xargs rm

[BASH Cures Cancer](http://bashcurescancer.com/)
