---
title: "Profiler week 5: Mac support, experiments profiling memory allocations"
juliasections: ['rbspy']
date: 2018-02-06T22:07:37Z
url: /blog/2018/02/06/profiler-week-5/
categories: []
---

Hello! Week 5 of profiler work is over. as a reminder -- what I've been doing is building a new
sampling CPU profiler for Ruby! It's called rbspy and it's at https://github.com/rbspy/rbspy.

In just-this-second news -- someone tweeted at me just now that they [used rbspy and it helped them find an unexpected hot spot in their program](https://twitter.com/nleach/status/961081617182703616), which made me super happy!!

The main 2 exciting things that happened last week were:

* **Mac support** is done! Supporting Mac is kind of an interesting thing because in my mind rbspy is a
  production profiler (for figuring out why your production server Ruby code is slow), and Macs
  are basically all laptops. Nonetheless! Supporting Mac is cool, people are using the Mac version (to [make perf improvements to CocoaPods, which is a Mac Ruby program](https://github.com/CocoaPods/CocoaPods/pull/7348#issuecomment-362002224)) and it's working and I'm happy about that.
* I've been doing a lot of **experiments with memory profiling**. Basically I'm curious about what
  memory profiling tools I can build that work from outside of the Ruby process (just give the tool
  a PID and it tells you about the internals of your Ruby program!)

### memory profiling experiment: tracking line numbers of every allocation

Here's an experiment I worked on today! I wanted to be able to answer the question -- which
functions are allocating memory right now?

So I wrote a small program that tracks every Ruby memory allocation, collects the function,
filename, and line number that that allocation happens at, and then aggregates the results! This
happens live, and should work on basically Ruby program with symbols.

here's the example output for a run of `rubocop` I did:

```
    635 wrap_with_sgr /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rainbow-3.0.0/lib/rainbow/string_utils.rb 13
    898 tok /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/parser-2.4.0.2/lib/parser/lexer.rb 21731
    931 visit_descendants /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/ast/node.rb 552
   1262 source /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/parser-2.4.0.2/lib/parser/source/range.rb 129
```

This says that there were 1262 allocations in the `source` function in `range.rb` on line 129. Let's
look at line 129, to see what that looks like:

```
128| def source
129|   @source_buffer.slice(self.begin_pos...self.end_pos) <---- line 129
130| end
```

It's not immediately obvious what the allocation is here, but it turns out that it's
`self.begin_pos...self.end_pos` -- that allocates a new `Range` object. So that's kind of
interesting!

This particular example doesn't let us make any actual performance improvements to rubocop (1000
allocations over 10 seconds is actually not a lot at all), but it's interesting to be able to see
which lines are allocating the most.

The code for that particular experiment is [here right now](https://github.com/rbspy/rbspy/blob/dda46eb0d97d54622b1a71c1edc468e991e8b996/src/malloc.rs). It's built using this [eBPF crate in Rust](http://crates.io/bcc) I've been working on.

### this week: debian packaging, more experiments

One goal for this week is to make a Debian PPA for rbspy -- I think that'll be really helpful
for Linux users (who are the main target audience I have in mind). There's also 1 bug with C stack
frames that I've been meaning to fix for a while. I feel like I'm mostly done (?) with core rbspy
functionality and that it's going to become time to focus more on UI and other non-CPU-profiler
experiments.

One UI concern that I have right now is -- flamegraphs are a nice way to present profiler data, but
they don't work well for recursive programs. So I might try to put together another visualization
style, like this style that pprof generates: (image taken from [the golang blog](https://blog.golang.org/profiling-go-programs)).

<div align="center">
<img src="https://blog.golang.org/profiling-go-programs_havlak1a-75.png">
</div>

I think for the rest of the sabbatical I'll split my time between stabilizing rbspy / adding UX
improvements to it and working on new experiments in memory profiling / other ways to observe what
Ruby programs are doing.

If you have ideas for experiments in the future of Ruby profiling, I'd love to hear them!

### want to help?

A few people have said they'd love to help with rbspy. In case that's you, a very useful thing you
can do is try it out and report your successes/issues!

I have a [tracking issue for success stories](https://github.com/rbspy/rbspy/issues/62) here, and
the github repo is https://github.com/rbspy/rbspy for reporting bugs/usability problems.
