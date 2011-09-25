--- 
layout: post
title: Log Levels
created: 1213829928
---
As a sysadmin I spend much of my day trawling though log files looking for problems.
Unfortunately many developers don't appear to know how to use log levels effectively which can make a sysadmin's life a small hell when it comes to debugging.

As a general rule I advise the following:
<blockquote>
Be quiet about successful operations and verbose about unsuccessful ones.
</blockquote>

Yes I like to see what an application is doing but I don't need to know every function call that it's making, yes this is useful when developing an application but can make isolating a problem in production a real pain.


Generally I use 4 different log levels:

<ul>
<li>DEBUG</li>
<li>INFO</li>
<li>WARN</li>
<li>ERROR</li>
</ul>

The definitions for these go something as follows:

<table>
<tr>
<td><b>DEBUG</b></td>
<td>Things that should happen but you don't want to see in production. This includes function tracing, SQL statements, variable values etc...</td>
</tr>
<tr>
<td><b>INFO</b></td>
<td>Things that should happen that give you just enough idea about what's going on. This should be limited to 1 or 2 lines per transaction,</td>
</tr>
<tr>
<td><b>WARN</b></td>
<td>Things that shouldn't happen but the end user should see no difference. For example $foo failed so falling back to $bar instead</td>
</tr>
<tr>
<td><b>ERROR</b></td>
<td>Houston we have a problem! This is a serious error that has caused a problem for the end user. If you want to dump variables to the logs then now is a good time to do it.</td>
</tr>
</table>

Some applications do logging in a completely different way, they have more log levels. Generally in production you want to know just enough to know what the application is doing + all the warnings and errors. 

If something goes wrong then you need enough information to be able to replicate it. If all is well then, as sysadmins, we don't care.

