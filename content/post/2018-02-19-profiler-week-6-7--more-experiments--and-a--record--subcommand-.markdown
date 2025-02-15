---
title: "Profiler week 6/7: more experiments, and a `report` subcommand!"
juliasections: ['rbspy']
date: 2018-02-19T23:20:25Z
url: /blog/2018/02/19/profiler-week-6-7--more-experiments--and-a--record--subcommand-/
categories: []
---

Hello! I didn't write a profiler blog post last week, but I am writing one today!

The most exciting thing that happened in the last few weeks is -- I think rbspy _works_. Like when I
started out on this sabbatical, I was pretty worried that I wouldn't be able to stabilize it and
that it would always be kind of buggy and unstable.

I think it works, though! I haven't gotten any new bug reports in a couple weeks, and all the bugs
I've run into so far have been quite easy to fix. There is an issue where the Mac version is a
little sketchy (because Mac systems programming is hard and I'm not planning to invest more time in
it), but I feel good about the Linux version!

I'm sure there are still bugs but it's surprising and exciting to me that this program actually
seems to be working for other people!! They can actually use it to profile their code!

### new feature: `rbspy report`

Recently I've mostly been working on rbspy user interface. I just merged a new feature called
`report`!

The idea is -- if you have a raw rbspy data file that you've previously recorded, you can use `rbspy
report` to generate different kinds of visualizations from it (the flamegraph/callgrind/summary
formats, as documented above). This is useful because you can record raw data from a program and
then decide how you want to visualize it afterwards!

For example, here's what recording a simple program and then generating a text summary report looks
like:

```
$ sudo rbspy record --raw-file raw.gz ruby ci/ruby-programs/short_program.rb
$ rbspy report -f summary -i raw.gz -o summary.txt
$ cat summary.txt
% self  % total  name
100.00   100.00  <c function> - unknown
  0.00   100.00  ccc - ci/ruby-programs/short_program.rb
  0.00   100.00  bbb - ci/ruby-programs/short_program.rb
  0.00   100.00  aaa - ci/ruby-programs/short_program.rb
  0.00   100.00  <main> - ci/ruby-programs/short_program.rb
```

I released it as v0.2.0! If you want to try it out as always you can download it on the github
releases page: https://github.com/rbspy/rbspy/releases

### new feature: always display a live summary

The other new feature I added recently (which I probably need to add a command line argument to
disable) is -- now rbspy will always display a live summary of the profiling data so far while
you're recording!

The idea here is that when you're recording profiling data it's useful to have a quick overview of
the big picture of what functions are taking up the most time. So rbspy calculates a summary and
updates it every second. Here's what the summary looks like: it's the top 20 functions (by self
time).

```
Summary of profiling data so far:
% self  % total  name
 19.38   100.00  <c function> - unknown
 11.52    29.07  block in tokens - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/cop/mixin/surrounding_s
 11.15    11.33  source_range - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/ast/node.rb
  5.48     5.48  end_pos - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/token.rb
  5.12    57.95  block (2 levels) in on_send - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/cop/commiss
  2.83    13.53  block in each_child_node - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/ast/node.rb
  2.74    14.17  each_child_node - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/ast/node.rb
  2.29     4.94  advance - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/parser-2.4.0.2/lib/parser/lexer.rb
  1.37     7.40  visit_descendants - /home/bork/.rbenv/versions/2.4.0/lib/ruby/gems/2.4.0/gems/rubocop-0.52.1/lib/rubocop/ast/node.rb
```

### new feature: track C functions

The last thing I added recently that's important is -- previously I just ignored C functions in the
ruby stack. I'm still not sure if it's possible to identify **which** C function is running, but I
put them back in. So now if there's a C function in the Ruby stack, it'll be reported as `<c
function> - unknown`, which is not extremely useful but is better than just ignoring it.

### next goal: release a beta, write some docs

My goal this week/next week is to release something I'm comfortable calling a "beta" release, and to
stop calling it experimental. Also I want to make a cool documentation site that teaches folks a
little bit about how profiling works and walks through some examples.

### experiments?

Last update I was really excited about some of the experiments I was doing with eBPF. I am still
excited about those but they're on pause for now because I decided it was better to actually finish
some stuff and write a documentation site. Maybe when I'm done that I can go back to the experiments
:)

Also next week I'm giving a talk about rbspy! Looking forward to that, but first I need to write the
talk :)
