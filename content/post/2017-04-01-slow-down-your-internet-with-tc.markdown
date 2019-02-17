---
title: "Slow down your internet with tc"
juliasections: ['Computer networking']
date: 2017-04-01T21:05:50Z
url: /blog/2017/04/01/slow-down-your-internet-with-tc/
categories: []
---

Hello! This week I learned how to make my internet slower on purpose! My
awesome coworker Doug told me about tc, which is a tool for Linux that
stands for "traffic control".

I'd actually heard about this tool a while ago when I interviewed
someone awesome and she was telling me about her experience using tc in
her last job. But I hadn't thought about it since then, and I definitely
hadn't used it!

He pointed me to this blog post [Adding simulated network latency to your Linux server](http://bencane.com/2012/07/16/tc-adding-simulated-network-latency-to-your-linux-server/)
which explains it really well.

### use tc to make your personal internet slow

I tried `tc` out on my laptop! The incantation in that blog post told
me how to add extra latency to every packet I send. So when any program
on my computer sends a network packet, it'll just get sent...  a bit
late. Here's what that looks like:

```
sudo tc qdisc add dev wlp3s0 root netem delay 500ms
# and turn it off with
sudo tc qdisc del dev wlp3s0 root netem
```

`wlp3s0` is the network interface for my wireless card, and this command
adds a 500ms delay to every packet. Totally unsurprisingly, browsing the
internet got a lot slower. There was something surprising, though!

Last week I learned about this site that compares the speed of HTTP and
HTTPS for loading a whole bunch of small images:
[http://www.httpvshttps.com/](http://www.httpvshttps.com/).

Here's a table with how long it took with my normal internet and with
artifically-slowed down internet.

```
                    http    | https
normal internet |  3.5s     | 0.7s
500ms delay     |  33s      | 1.1s
(with TLS connection already open)
500ms delay     |  33s      | 5s
(with new TLS connection)
```

That's weird, right? The HTTP version got 10x slower, but the https
version didn't slow down as much. To understand why, you have to
read this article [I wanna go fast: HTTPS' massive speed advantage ](https://www.troyhunt.com/i-wanna-go-fast-https-massive-speed-advantage/).

Basically HTTP/2 (the next version of HTTP) only
works when you're using HTTPS, and HTTP/2 lets you do a lot more
requests in parallel, so you're not slowed down as much.

There's something else weird in this table though. It says that with my
500ms delay, the HTTPS version of the page loaded in 1.1s. But to do HTTPS you have to do a
TLS handshake, and if your latency is 500ms you can definitely not 
open the connection and send/receive data in 1.1s. When the HTTPS version was taking 1.1s it
was reusing a TLS connection that was already open. Still, when I
restarted my browser and opened the website from scratch it only took 5
seconds. That's really a lot faster than 33s! I might conceivably wait 5
seconds for a page to load but I feel like I'd get too bored in 30 seconds.

So it's really neat to see that HTTPS and HTTP/2 are really possibly
making things faster for people with high latency!


### use tc to make your server internet slow

You can also use tc to make your server internet slow. I used tc to make
a DNS server in a test environment slow to see how a DNS client I was
interested in would behave, exactly.

It's important to note that -- you probably don't want to add a
delay to every packet on a machine in production, you might have a bad
day. Unless your goal is really to break things  ("surprise, network
latency everyone! let's find out how your code deals with timeouts!") in
which case maybe it's a good idea.

One thing I learned by doing this -- if you add a delay to every
packet on a machine you're SSHed into, your SSH connection is also
sending packets, so your SSH connection also gets slow. This was okay
though, it was still usable enough.

### that's all

I think there are more things you can do with tc but I don't know what
they are yet! I think it's cool that you can pretend to be on a machine
that is Really Far Away by just running one command on your laptop!
