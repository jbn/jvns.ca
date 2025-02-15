---
categories: ["talks"]
juliasections: ['Talks transcripts / podcasts']
comments: true
date: 2016-09-17T15:32:50Z
title: 'A swiss army knife of debugging tools: talk & transcript'
url: /blog/2016/09/17/strange-loop-talk/
---

<style>

.container {
	display: flex;
}
.slide {
	width: 50%;
}
.content {
	width: 50%;
	align-items: center;
	padding: 20px;
}

@media (max-width: 480px) { /*breakpoint*/
    .container {
        display: block;
    }
    .slide {
    	width: 100%;
    }
    .content {
    	width: 100%;
}

</style>

Yesterday I gave a talk at Strange Loop. I'll try to write more about the conference and my favorite things about it later, but for now here's the talk I gave.

### video

<iframe width="560" height="315" src="https://www.youtube.com/embed/HfD9IMZ9rKY" frameborder="0" allowfullscreen></iframe>

### transcript

I mean "transcript" in a very loose sense here -- this is pretty approximate. I wrote it all down without watching the actual talk I gave at all.


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_02.png"><img src="/images/stl-talk/slide_02_small.png"></a>
</div>
<div class="content">
<p>
Hi! I'm Julia. I work at Stripe and today I'm going to tell you about some of my favorite debugging tools!
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_03.png"><img src="/images/stl-talk/slide_03_small.png"></a>
</div>
<div class="content">
<p>
an alternate title slide!
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_04.png"><img src="/images/stl-talk/slide_04_small.png"></a>
</div>
<div class="content">
<p>
At Strange Loop we often talk about programming languages! This talk isn't about programming languages at all. Instead, we're going to talk about debugging tools that you can use to debug programs in <i> any </i> programming language. 
</p>
<p>
When Haskell does networking, it uses TCP. Python uses TCP! Fortran! So if we debug programs using networking tools, we can debug programs in any language.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_05.png"><img src="/images/stl-talk/slide_05_small.png"></a>
</div>
<div class="content">
<p>
When I log into a computer, sometimes something has gone TERRIBLY WRONG. And in these situations, sometimes you can feel helpless! We're going to talk about tools you can use to figure out what has happened.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_06.png"><img src="/images/stl-talk/slide_06_small.png"></a>
</div>
<div class="content">
<p>
I used to think to debug things, I had to be really really smart. I thought I had to stare at the code, and think really hard, and magically intuit what the bug was.
</p>
<p>
It turns out that this isn't true at all! If you have the right tools, fixing bugs can be *really easy*. Sometimes with just a little more information, you can figure out what's happening without being a genius at all.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_07.png"><img src="/images/stl-talk/slide_07_small.png"></a>
</div>
<div class="content">
<p>
The last thing I want to encourage you to do before we start this talk is -- we're going to be talking about a lot of systems tools. It's easy to think "oh, this is operating systems, it's too hard."
</p>
<p>
But if you're not scared, you can usually figure out almost anything!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_08.png"><img src="/images/stl-talk/slide_08_small.png"></a>
</div>
<div class="content">
<p>
So, when we normally debug, we usuually read the code, add print statements, and you should probably know the programing language of the program you're writing, right?
</p>
</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_09.png"><img src="/images/stl-talk/slide_09_small.png"></a>
</div>
<div class="content">
<p>
Nope! This isn't true at all. You can totally debug programs without having their source code or even knowing what language they're written in at all. That's what we're going to do in this talk.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_10.png"><img src="/images/stl-talk/slide_10_small.png"></a>
</div>
<div class="content">
<p>
Here are some of the tools we're going to discuss in this talk! strace, ngrep, wireshark, etc.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_11.png"><img src="/images/stl-talk/slide_11_small.png"></a>
</div>
<div class="content">
<p>
And the way that most of these work is like the following -- you have a question ("what file did my program open"), you use a tool to interrogate your operating system about what the program is doing, and hopefully the answer you get back helps you fix your problem.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_12.png"><img src="/images/stl-talk/slide_12_small.png"></a>
</div>
<div class="content">
<p>
I'm going to explain how these tools work through a series of mysteries (the case of the missing configuration file! the case of the slow program! the case of the French website!) and then I'm going to go a little more in depth into two more tools -- perf and tcpdump.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_13.png"><img src="/images/stl-talk/slide_13_small.png"></a>
</div>
<div class="content">
<p>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_14.png"><img src="/images/stl-talk/slide_14_small.png"></a>
</div>
<div class="content">
<p>
Who's written a configuration file and been confused about why it didn't work and realized they were editing the WRONG FILE? Yeah, me too. This is really annoying! 
</p>
<p>
Normally to figure out what the right configuration file is for your program, you might read the documentation or as a coworker. But what if the documentation is wrong, or your coworker doesn't know? What do you do if you want to be really sure?
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_15.png"><img src="/images/stl-talk/slide_15_small.png"></a>
</div>
<div class="content">
<p>
A really classic example of this is .bashrc vs .bash_profile -- when you start bash, which configuration file does it use? How can you tell? I actually know this through experience, but what if I didn't?
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_16.png"><img src="/images/stl-talk/slide_16_small.png"></a>
</div>
<div class="content">
<p>
To figure this out, we're going to use my absolute favorite program! STRACE. strace traces system calls.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_17.png"><img src="/images/stl-talk/slide_17_small.png"></a>
</div>
<div class="content">
<p>
Let's talk about what a system call is. Your program does not itself know how to open files. To open a file, you need to understand how your hard drive works, and what filesystem is on that hard drive, and all kinds of complicated things.
</p>
<p>
Instead, your program asks the operating system to open a file. One that file is open, it can ask the OS to read and write to the file. The words I've underlined in red (open and read) are called **system calls**.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_18.png"><img src="/images/stl-talk/slide_18_small.png"></a>
</div>
<div class="content">
<p>
strace will tell you every system call a program is running. To run strace on a program, you just say "strace" before it.
</p>
<p>
When I run strace on bash and ask it to look at just "open" system calls, I can see that it's opening .bashrc! Great! We win.
</p>

</div>
</div>




<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_20.png"><img src="/images/stl-talk/slide_20_small.png"></a>
</div>
<div class="content">
<p>
So we answered our question! Awesome.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_19.png"><img src="/images/stl-talk/slide_19_small.png"></a>
</div>
<div class="content">
<p>
Before we move on, I want to show you a quick demo of what it actually looks like to use strace. (do demo>
</p>
<p>
What you see is that strace prints approximately a billion lines of output, and you very very likely don't know what they all mean. This is okay!
</p>
<p>
When I use strace, I ignore practically everything, and just grep to find the one thing I'm interested in. This makes it a lot easier to understand :)
</p>

</div>
</div>

<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_21.png"><img src="/images/stl-talk/slide_21_small.png"></a>
</div>
<div class="content">
<p>
An extremely important thing to know about strace is that if you run it on a program, it can make that program run up to 50x slower, depending on how many system calls that program uses.
</p>
<p>
So don't run it on your production database.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_22.png"><img src="/images/stl-talk/slide_22_small.png"></a>
</div>
<div class="content">
<p>
Here are some of my favorite system calls! There are system calls for communicating over the network (what computers is my program talking to?), for reading and writing files, and executing programs.
</p>
<p>
execve is one of my favorites because -- sometimes I write scripts that run a bunch of other programs. If the script is doing the wrong thing, it can be really annoying to debug! Reading code is a lot of work.
</p>
<p>

But if you use strace, you can just see the commands that got run really quickly, see what is wrong, and then go back and track it down in your program to fix it.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_23.png"><img src="/images/stl-talk/slide_23_small.png"></a>
</div>
<div class="content">
<p>
Some really important command line flags to strace! -f lets you strace the process and every subprocess it creates.  I basically always run strace with -f.
</p>
<p>
-y is an amazing flag in new versions of strace that shows you the filename of the file you're reading to and writing from. (instead of just the file descriptor)
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_24.png"><img src="/images/stl-talk/slide_24_small.png"></a>
</div>
<div class="content">
<p>
I was so excited when I learned about strace, and I couldn't believe that I'd been programming for 9 years without knowing about it. So I wrote a zine about my love for strace. You can find it at <a href="http://jvns.ca/zines">jvns.ca/zines</a>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_25.png"><img src="/images/stl-talk/slide_25_small.png"></a>
</div>
<div class="content">
<p>
Okay, so I told you that strace is slow! What if you want something that is not slow? If you're on a Linux kernel version above 4.4 or so, you're in luck. There's a set of tools you can download from <a href="https://github.com/iovisor/bcc">https://github.com/iovisor/bcc</a>, which include something called "opensnoop".
</p>
<p>
(do opensnoop demo). Basically opensnoop can tell you which files your programs are opening, but without slowing down your programs! amazing!
</p>
<p>
In particular Ubuntu 16.04 is new enough for this tool.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_26.png"><img src="/images/stl-talk/slide_26_small.png"></a>
</div>
<div class="content">
<p>
Opensnoop (and the other scripts in that repo I linked to) work using eBPF. <a href="http://www.brendangregg.com/"> Brendan Gregg</a> has been writing a lot about eBPF for a while. It seems super interesting.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_27.png"><img src="/images/stl-talk/slide_27_small.png"></a>
</div>
<div class="content">
<p>
Okay, next mystery!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_28.png"><img src="/images/stl-talk/slide_28_small.png"></a>
</div>
<div class="content">
<p>
Here I'm going to discuss 3 slow programs, which are all slow for different reasons. We're going to figure out why they're all slow, without reading the source code or anything. All the programs are written in Python.
</p>
<p>
They're slow because of CPU time, writing too much to disk, and because of waiting for a reply from a network connection.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_29.png"><img src="/images/stl-talk/slide_29_small.png"></a>
</div>
<div class="content">
<p>
Here's the first one!
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_31.png"><img src="/images/stl-talk/slide_30_small.png"></a>
</div>
<div class="content">
<p>
I'm going to run `time` on all these programs. time is a nice program! It tells you how long the program took (2 seconds), but that's not all!
</p>
<p>
It also breaks down how the time was spent. This program spent 5% of its time on the CPU! So for the remaining 95% of the time it was waiting.
</p>

</div>
</div>

<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_32.png"><img src="/images/stl-talk/slide_32_small.png"></a>
</div>
<div class="content">
<p>
The program could have been waiting for a lot of different things! Network, disk, just because it decided to hang out and wait?
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_33.png"><img src="/images/stl-talk/slide_33_small.png"></a>
</div>
<div class="content">
<p>
Luckily this is pretty easy to find out! We can peer into the Linux kernel's soul and figure out what it was doing when the program was waiting.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_34.png"><img src="/images/stl-talk/slide_34_small.png"></a>
</div>
<div class="content">
<p>
For any program, we can take the program's PID (in this case 31728), and ask what the Linux kernel is doing for that program right now.
</p>
<p>
We get a call stack starting with the system call and ending up with the current function. Awesome!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_35.png"><img src="/images/stl-talk/slide_35_small.png"></a>
</div>
<div class="content">
<p>
To help you see what's going on, I deleted almost everything. I know what tcp_recvmsg means! It means it's waiting to receive a message on a TCP connection!
</p>
<p>
That's networking! That was really easy! We don't need to be kernel debugging wizards to figure out what's going on.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_36.png"><img src="/images/stl-talk/slide_36_small.png"></a>
</div>
<div class="content">
<p>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_37.png"><img src="/images/stl-talk/slide_37_small.png"></a>
</div>
<div class="content">
<p>
This was the actual program that the server was running -- you can see that it sleeps for 2 seconds, and then returns a response of "hi!".
</p>
<p>
So it's obvious why the program was slow :) But you can apply the exact same technique to much more complicated programs.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_38.png"><img src="/images/stl-talk/slide_38_small.png"></a>
</div>
<div class="content">
<p>
</p>
<p>
</p>

</div>
</div>

<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_40.png"><img src="/images/stl-talk/slide_40_small.png"></a>
</div>
<div class="content">
<p>
When we run `time` on our next program, we see something really interesting right away! It's spending 99% of its time just using the CPU.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_41.png"><img src="/images/stl-talk/slide_41_small.png"></a>
</div>
<div class="content">
<p>
At this point we're actually done -- since this is a Python program, the easiest thing to do is probably just to run a Python profiler to find out what it was doing.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_42.png"><img src="/images/stl-talk/slide_42_small.png"></a>
</div>
<div class="content">
<p>
And what it was actually was adding up 14 million numbers. You can decide whether you think 2.74 seconds is how long you think it should take to add up 14 million numbers or not :)
</p>
<p>
(I made a whole game about this called <a href="http://computers-are-fast.github.io/">computers are fast </a>)
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_43.png"><img src="/images/stl-talk/slide_43_small.png"></a>
</div>
<div class="content">
<p>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_44.png"><img src="/images/stl-talk/slide_44_small.png"></a>
</div>
<div class="content">
<p>
Okay, this one is spending 94% of its time on the CPU. So this is basically the same as before, right? Nope!
</p>
<p>
You'll notice that there are two kinds of CPU your program can use: <b> user</b> CPU and <b> system</b> CPU. This one is spending most of its time on CPU, but CPU in the kernel! So your program is still spending most of its time waiting.
</p>
<p>
</p>
</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_45.png"><img src="/images/stl-talk/slide_45_small.png"></a>
</div>
<div class="content">
<p>
If you have a program that's waiting, a nice way to try to figure out what's going on is to use <b> dstat </b>
</p>
<p>
dstat is a nice little program that prints our how much network, disk, and CPU your program is using 
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_46.png"><img src="/images/stl-talk/slide_46_small.png"></a>
</div>
<div class="content">
<p>
Here I do a demo where I run a program (python mystery_3.py), and run `dstat` while the program is running.
</p>
<p>
dstat is shows that while our program is running, 300MB/s get written to disk. When the program stops, the disk writes stop. Interesting!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_47.png"><img src="/images/stl-talk/slide_47_small.png"></a>
</div>
<div class="content">
<p>
So we understand that something's going on with writes, but why is it using all this CPU? Surely disk writes don't use that much CPU?
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_48.png"><img src="/images/stl-talk/slide_48_small.png"></a>
</div>
<div class="content">
<p>
Who's used top? (everyone) htop? (a few less people) perf top? (almost nobody)
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_49.png"><img src="/images/stl-talk/slide_49_small.png"></a>
</div>
<div class="content">
<p>
So, top tells you which one of your programs is using all your CPU. That's great.
</p>
<p>
But it doesn't tell you which *functions* your programs are running. `perf top` does, though! Let's see what that looks like.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_50.png"><img src="/images/stl-talk/slide_50_small.png"></a>
</div>
<div class="content">
<p>
I run `python mystery_3.py` again, and I quickly switch to another terminal tab and run `perf top`. perf top shows a bunch of functions and how much CPU time each one is using.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_51.png"><img src="/images/stl-talk/slide_51_small.png"></a>
</div>
<div class="content">
<p>
There was a lot to look at one the screen, so let's zoom in. The top function is something called `_aesni_enc1`. What does AES tell us? ENCRYPTION! That's right!
</p>
<p>
So it turns out that this program is writing files to disk, and specifically it's writing to my home directory. But my home directory is encrypted! So it needs to spend a bunch of extra time encrypting all the data, and that's what all the CPU time is about. Awesome.
</p>
<p>
So we've solved our mystery!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_52.png"><img src="/images/stl-talk/slide_52_small.png"></a>
</div>
<div class="content">
<p>
One really awesome thing about perf is -- normally perf only tells you about which *C functions* are running on your computer. But you might want to know about which functions in your programming language are running on your computer! In Java, you can use <a href="https://github.com/jrudolph/perf-map-agent">perf-map-agent</a> to get perf to tell you what Java functions are running on your computer!
</p>
<p>
And you can do the same thing <a href="http://www.brendangregg.com/blog/2014-09-17/node-flame-graphs-on-linux.html">with node.js</a>.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_53.png"><img src="/images/stl-talk/slide_53_small.png"></a>
</div>
<div class="content">
<p>
Our last performance program we're going to look at is the case of the French website.
</p>
<p>
I live in Montreal, and it's a bilingual city, so when you open a city website, it might show up in either French or English. You never know! What determines that?
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_54.png"><img src="/images/stl-talk/slide_54_small.png"></a>
</div>
<div class="content">
<p>
This is a website I made for this conference. It just says "hello! welcome to strange loop!" Great.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_55.png"><img src="/images/stl-talk/slide_55_small.png"></a>
</div>
<div class="content">
<p>
But when I get the very same website from my terminal, it replies in French, not in English! What's going on!
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_56.png"><img src="/images/stl-talk/slide_56_small.png"></a>
</div>
<div class="content">
<p>
Most of you probably use grep, which lets you search text files. ngrep is a program that lets you search your network traffic!
</p>
<p>
Here I do a demo where I use ngrep to show all TCP packets on localhost. We
see that with curl, there's a very very simple HTTP request, but Google Chrome
has a much longer request and a lot more to share with the server. In
particular, the request has the string "Accept-Language: en-US" in it. When we
add that header to our curl request, we get a response in English! Awesome.
</p>
<p>

Computer programs aren't always deterministic, but they're always logical. If
you're seeing a difference where you think there shouldn't be any, there's
always a reason. Looking at the inputs and outputs to your programs can help you figure out what's going on, and network traffic is a really nice place to do that.

</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_57.png"><img src="/images/stl-talk/slide_57_small.png"></a>
</div>
<div class="content">
<p>
Okay, so this is a totally new section of the top.
</p>
<p>
Here we're going to talk about two of my favorite things: perf and tcpdump.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_58.png"><img src="/images/stl-talk/slide_58_small.png"></a>
</div>
<div class="content">
<p>
perf is a profiling tool for Linux that a lot of people don't know about, mostly because it's really sparsely documented. My goal today is to change that a little bit.
</p>
<p>
One really confusing thing about perf is that it actually has 3 almost totally unrelated tools in them: it can tell you about your hardware counters, do sampling profiling for CPU, and trace events. I'm going to talk about all these 3 things separately.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_59.png"><img src="/images/stl-talk/slide_59_small.png"></a>
</div>
<div class="content">
<p>

Before we move on, I want to talk about sampling vs tracing for a little bit.
In a sampling profiler, you look at a small percentage of what's going on in
your program (what's the stack trace now? how about now?), and then use that information to generalize about what your program is doing.

</p>
<p>
This is great because it reduces overhead, and usually gives you a good idea of what your program is doing.
</p>
<p>
But what happens if you have an error that happens infrequently? like 1 in 1000 times. You might think this is not a big deal, and in fact for a lot of people that might not be a big deal.
</p>
<p>

But I write programs that do things millions or billions of times, and I want
those things to be really highly reliable, so even relatively rare events are
important to me. So I love tracing tools and log files that can tell me
**everything** about when a certain function is called. This makes it a lot easier to debug than just having a general statistical distribution! <br> This is a great <a href="http://danluu.com/perf-tracing/">post about tracing tools</a> by Dan Luu.
</p>


</div>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_60.png"><img src="/images/stl-talk/slide_60_small.png"></a>
</div>
<div class="content">
<p>
Let's start with hardware counters. The story with these is that your computer's hardware has a lot of events it can count and keep track of for you, if you just ask it to.
</p>
<p>
We're going to talk about one small thing you might like to count: L1 cache misses.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_61.png"><img src="/images/stl-talk/slide_61_small.png"></a>
</div>
<div class="content">
<p>
Very very close to your CPU, there is a small data cache called the L1 cache. It's very fast to access the L1 cache, maybe 0.5 nanoseconds. But if you want to get data from RAM, it's slow! 200x slower. 
</p>
<p>
So if you're accessing data and working with on it on the CPU, and trying to do this in a high performance way, you care about whether your data is coming from a CPU cache or from RAM.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_62.png"><img src="/images/stl-talk/slide_62_small.png"></a>
</div>
<div class="content">
<p>
There are a ton of these numbers that are interesting to know -- there's a famous gist file called "latency numbers every programmer should know"
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_63.png"><img src="/images/stl-talk/slide_63_small.png"></a>
</div>
<div class="content">
<p>
Here it is.
</p>
<p>
I don't actually have all these numbers memorized, but I do find this useful to remember a few things from it -- if I want something to happen in a microsecond, I know I absolutely cannot have a network request happening, for example.
</p>
<p>
But all of thse numbers seemed really abstract to me for a long time. How can you even know whether or not your program is using a L1 cache? That's like inside your computer! I can't see inside my computer!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_64.png"><img src="/images/stl-talk/slide_64_small.png"></a>
</div>
<div class="content">
<p>

But you can! `perf stat` will let you look at all kinds of counters, and in
particular it can show you how many L1 cache misses you have. Here we have `ls` and we can see that there were about 40,000 cache misses. So this is something you can totally measure.

</p>
<p>
I wrote a blog post about this in <a href="http://jvns.ca/blog/2014/05/13/profiling-with-perf/">I can spy on my CPU cycles with perf!</a>

</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_65.png"><img src="/images/stl-talk/slide_65_small.png"></a>
</div>
<div class="content">
<p>
Next up, let's talk about sampling profilers! So there's a system in the Linux kernel called `perf_events`, where you can ask it "hey, keep track of what my program is doing for me, okay?". And it'll collect statistics and report them back to you. This is how the program `perf top` works that I showed you earlier.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_66.png"><img src="/images/stl-talk/slide_66_small.png"></a>
</div>
<div class="content">
<p>
To ask Linux to start recording performance information for you, you run `perf record` on your program. The -g flag is really great because it lets you collect full stack traces for everything that's happening.
</p>
<p>
And then you can generate a nice report with `sudo perf report`!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_67.png"><img src="/images/stl-talk/slide_67_small.png"></a>
</div>
<div class="content">
<p>
Here's what that report looks like. This is from recording how the program `find` spends its time.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_68.png"><img src="/images/stl-talk/slide_68_small.png"></a>
</div>
<div class="content">
<p>
If we zoom in, we can see all these `getdents` functions. This makes sense because find spends all its time searching through directories, and `getdents` is how you list a directory on Linux.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_69.png"><img src="/images/stl-talk/slide_69_small.png"></a>
</div>
<div class="content">
<p>
Now let's talk about an awesome way to look at these  performance reports: flame graphs. These are basically a way to visualize stack traces: `main` is at the bottom, and of course you spend all your time there, and as you go up you see how the time is split up between different inner functions.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_70.png"><img src="/images/stl-talk/slide_70_small.png"></a>
</div>
<div class="content">
<p>
Here's an artisanal flame graph I drew to help explain what these flame graphs mean exactly! 
</p>
<p>
Basically we start out in main, and then 20% of the time we go to `panda` and 80% of the time we go to alligator. And so on.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_71.png"><img src="/images/stl-talk/slide_71_small.png"></a>
</div>
<div class="content">
<p>
flamegraphs aren't just for perf: you can also generate them from other tools. The software I used to make this graph is on github, and by Brendan Gregg: <a href="https://github.com/brendangregg/FlameGraph">https://github.com/brendangregg/FlameGraph</a>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_72.png"><img src="/images/stl-talk/slide_72_small.png"></a>
</div>
<div class="content">
<p>
So, let's talk about the last part of this: tracing. </p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_73.png"><img src="/images/stl-talk/slide_73_small.png"></a>
</div>
<div class="content">
<p>
We've already talked about two ways to trace events: strace traces system calls (but is slow), opensnoop and other eBPF-based tools do tracing (but aren't always available). The eBPF stuff is actually more powerful than `perf trace`, but I think `perf trace` might be a little more widely available.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_74.png"><img src="/images/stl-talk/slide_74_small.png"></a>
</div>
<div class="content">
<p>
I can run `perf trace` on my laptop like this.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_75.png"><img src="/images/stl-talk/slide_75_small.png"></a>
</div>
<div class="content">
<p>
And here's what the output looks like! We see a lot of system calls. This is kind of unfortunate, though -- it's actually dropping a lot of events. I think what's happening here is -- perf's kernel component writes events into a ring buffer in userspace, and I think that buffer's maybe filling up. So writing a tracing system that doesn't drop events apparently isn't trivial! Interesting.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_76.png"><img src="/images/stl-talk/slide_76_small.png"></a>
</div>
<div class="content">
<p>
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_77.png"><img src="/images/stl-talk/slide_77_small.png"></a>
</div>
<div class="content">
<p>
Okay, so we learned about ngrep earlier and how we can use it to find out why a website is in French.
</p>
<p>
ngrep is an amazing starter tool (and it can actually do a lot!), but what if you want to do something that you can't do with ngrep? Let's talk about tcpdump and wireshark!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_78.png"><img src="/images/stl-talk/slide_78_small.png"></a>
</div>
<div class="content">
<p>
It took me a long time to learn these two tools, but now I think they're pretty useful.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_79.png"><img src="/images/stl-talk/slide_79_small.png"></a>
</div>
<div class="content">
<p>
(demo of running tcpdump)
</p>
<p>
When you run tcpdump, it's originally pretty scary -- you end up seeing all this stuff you don't understand. But it turns out that it's totally okay! You can actually just use tcpdump to record the network traffic you're interested in analyzing, and then put it on your laptop and analyze it with some more user-friendly tools. 
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_80.png"><img src="/images/stl-talk/slide_80_small.png"></a>
</div>
<div class="content">
<p>
When you run tcpdump, you can always use a filter. `perf 80` is called a "berkeley packet filter", which is a small language that gets compiled to a virtual machine that lets you really quickly filter network traffic.
</p>
<p>
</p>

</div>
</div>

<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_82.png"><img src="/images/stl-talk/slide_82_small.png"></a>
</div>
<div class="content">
<p>
The BPF filter language (at least at a basic level) is pretty easy to learn -- you can filter by IP address and port, and those two things by themselves will already take you a long way. 
</p>
<p>
I don't really know more than that and that's good enough for me.
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_81.png"><img src="/images/stl-talk/slide_81_small.png"></a>
</div>
<div class="content">
<p>
When you write out network data with tcpdump, it goes into a "pcap file".
</p>
<p>
Basically every network analysis tool understand pcap files, so this is awesome.
</p>

</div>
</div>





<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_83.png"><img src="/images/stl-talk/slide_83_small.png"></a>
</div>
<div class="content">
<p>
You can also just print out tcp packets to your screen with `tcpdump -A`. I do this sometimes when I'm like ARE THERE EVEN PACKETS COMING INTO THIS MACHINE. Usually this is when I find out I've gotten my firewall settings wrong.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_84.png"><img src="/images/stl-talk/slide_84_small.png"></a>
</div>
<div class="content">
<p>
Okay, so you've recorded some network data with tcpdump, and you want to analyze it!
</p>
<p>
We're going to look at it with Wireshark!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_85.png"><img src="/images/stl-talk/slide_85_small.png"></a>
</div>
<div class="content">
<p>
this is what it looks like when you open wireshark
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_86.png"><img src="/images/stl-talk/slide_86_small.png"></a>
</div>
<div class="content">
<p>
this can be scary to look at, kind of like tcpdump, because there is all this INFORMATION and what does it even MEAN
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_87.png"><img src="/images/stl-talk/slide_87_small.png"></a>
</div>
<div class="content">
<p>
But Wireshark is not actually impossible to use. It has a really good filtering language and a lot of things to click on. The UI is pretty reasonable, though you do have to ignore a lot of stuff you don't understand.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_88.png"><img src="/images/stl-talk/slide_88_small.png"></a>
</div>
<div class="content">
<p>
For example! I wanted to filter this network traffic so I searched for all the GET requests. Not that hard!
</p>
<p>
here we see that there are only 5 packets here, I can see where they came from and where they went to, and what time they happened at. Neat! 
</p>
<p>
Wireshark also understand a ton of common network protocols, which is amazing. It understand simple stuff like HTTP, but also more complicated stuff like the MongoDB protocol!  So if you recorded Mongo traffic, which uses a binary protocol, Wireshark will know things like "This is the database table you were querying!"
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_89.png"><img src="/images/stl-talk/slide_89_small.png"></a>
</div>
<div class="content">
<p>
If you click on 'Statistics -> Conversations', you can also see how long each of the TCP conversations took, which is super awesome.
</p>
<p>
It's like.. Who are you? Are you a magician!?
Yes, I'm Wireshark! I'm a magical shark!
Good thing you're on my side!
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_90.png"><img src="/images/stl-talk/slide_90_small.png"></a>
</div>
<div class="content">
<p>
So all this is to say -- if you learn your operating systems, there are a ton of awesome tools available to you to understand your programs. 
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_91.png"><img src="/images/stl-talk/slide_91_small.png"></a>
</div>
<div class="content">
<p>
And a lot of these tools are totally possible to use! My favorite thing in the world is when people tell me "hi julia! I used this tool you told me about and now my boss thinks I'm a genius." This could be you!
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_92.png"><img src="/images/stl-talk/slide_92_small.png"></a>
</div>
<div class="content">
<p>
I think all this is really important and often poorly documented, so I wrote <a href="http://jvns.ca/zines/#linux-debugging-tools">a zine explaining a lot of these tools</a>, which gives simple summaries of what all of them do. You can read it and print it out and give it to your friends.
</p>
<p>
</p>

</div>
</div>


<div class="container">
<div class="slide">
<a href="/images/stl-talk/slide_93.png"><img src="/images/stl-talk/slide_93_small.png"></a>
</div>
<div class="content">
<p>
Thanks for listening.
</p>
<p>
</p>

</div>

