---
categories: ["hackerschool"]
juliasections: ['Recurse Center']
comments: true
date: 2013-09-30T00:00:00Z
title: 'Day 1: What does a shell even do?'
url: /blog/2013/09/30/hacker-school-day-2-what-does-a-shell-even-do/
---

So I'm working on writing a shell in C a little bit. Before yesterday, I
didn't have a very clear idea of what writing a shell even *meant*. Here
are some things that your shell has to do! I'm sure there are some
Important Things missing.

* Parse what you type in to figure out which are the commands and which
  are the arguments (`ls -la LICENSE`)
* expand `ls *` into `ls file1 file2 file3 ...`
* Pipes! If you write `ls | grep blah`, it needs to send the output from
  `ls` into `grep`. And redirection too.
* Signal handling! If you press `Ctrl+C`, it needs to send that signal
  to whatever process you're running. Or something. I don't really
  understand this yet.
* Process management! Lets you background and foreground jobs. (`Ctrl+z`
  and `fg` and `bg`)
* Shell scripting! (for loops and things)

And a thing the shell *doesn't* have to do:

* Figure out which command to execute using the `$PATH` environment
  variable. `exec` does that, apparently.

I think I'm going to work on implementing pipes & redirection & signal
handling.
