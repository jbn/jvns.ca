---
categories: ["hackerschool"]
juliasections: ['Recurse Center']
comments: true
date: 2013-10-01T00:00:00Z
title: 'Day 2: netcat fun!'
url: /blog/2013/10/01/day-2-netcat-fun/
---

Today [Alan](https://github.com/happy4crazy) taught me some things about
networking. IN PARTICULAR that you can transfer a file to your friend on
your local network with netcat.

Also it is pretty fun to watch the transfer with
[Wireshark](http://www.wireshark.org/).

Here's how it works:

I run

`netcat -l 12345 > file.pdf` or `netcat -l -p 12345 > file.pdf`

depending on my version of netcat. (BSD vs not-BSD or something)

You run `netcat $MY_IP_ADDRESS 12345 < file.pdf`

Then you wait a while until you think it's probably done and stop it.
And I have the file! That is all. Since you can see the whole thing in
Wireshark, it is not secure or anything and anyone in the middle could
also get the file. FUN.
