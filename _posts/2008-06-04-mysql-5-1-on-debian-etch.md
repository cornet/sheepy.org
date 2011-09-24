--- 
layout: post
title: MySQL 5.1 on Debian Etch
created: 1212611297
---
Here is a quick guide to getting MySQL installed on Debian Etch without pulling
in anything from testing or unstable.

Now there are a number of different ways you could do this but I wanted a debian
package (as I have banned installing anything from source on new servers at work).

After some google action I found that <a href="http://tretkowski.de/blog/">Norbert Tretkowski</a> had submitted packages to <a href="http://packages.debian.org/experimental/misc/mysql-server">experimental</a>
However after checking these out they wanted to pull things in from testing and/or unstable and I didn't want
that (especially considering it wanted to upgrade libc6)

That said it turned out that this package would build fine on Etch by doing the following:

Add the following to <em>/etc/apt/sources.list</em>:

    deb http://ftp.uk.debian.org/debian/ experimental main contrib non-free
    deb-src http://ftp.uk.debian.org/debian/ experimental main contrib non-free


Then create <em>/etc/apt/preferences</em> with the following in:

    Package: *
    Pin: release o=Debian,a=stable
    Pin-Priority: 900
    Package: *
    Pin: release o=Debian
    Pin-Priority: -1
 
which basically says "only install stuff from stable".


Next we get the dev tools and the build dependencies:

    aptitude install dpkg-dev devscripts
    apt-get build-dep mysql-server-5.1
    apt-get source mysql-server-5.1

Then build the .debs:

    cd mysql-dfsg-5.1-5.1.24rc
    debuild 


Once that has finished get the run time deps:

    aptitude install libdbd-mysql-perl libdbi-perl


And finally install the .debs we created earlier:

    cd ..
    dpkg -i *.deb

...and there you go, MySQL 5.1 on Debian Etch :)
