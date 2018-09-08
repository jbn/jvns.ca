---
title: "An awesome new Python profiler: py-spy!"
date: 2018-09-08T13:09:05Z
url: /blog/2018/09/08/an-awesome-new-python-profiler--py-spy-/
categories: []
---

The other day I learned that [Ben Frederickson](https://twitter.com/benfrederickson) has written an awesome new profiler
called [py-spy](https://github.com/benfred/py-spy).

It takes a similar approach to profiling as [rbspy](https://rbspy.github.io), the profiler I worked
on earlier this year -- it can profile any running Python program, it geenit uses process_vm_readv to read
memory, and it by default displays profiling information in a really easy-to-use way.

Obviously, think this is SO COOL. Here's what it looks like profiling a Python program: (gif taken
from the github README)

<img src="https://raw.githubusercontent.com/benfred/py-spy/8ea64fae73b746a5167798d9dc46e24939d395eb/images/console_viewer.gif">

It has this great top-like output by default. It's similar to what rbspy does, but feels like a much
nicer user experience to me.

### you can install it with pip!

Another thing he's done that's really nice is make it installable with `pip` -- you can run `pip install
py-spy` and have it download a binary immediately! This is cool because, even though `py-spy` is a
Rust program, obviously Python programmers are used to installing software with `pip` and not
`cargo`, so making it installable with pip is important for adoption.

In [the README](https://github.com/benfred/py-spy) he describes what he had to do to distribute a
Rust executable with pip without requiring that users have a Rust compiler installed.


### pyspy probably is more stable than rbspy!

One nice thing about building a Python profiler is that I believe it only uses Python's public
bindings (eg `Python.h`). What I mean by "public bindings" is the header files you'd find in
[libpython-dev](https://packages.ubuntu.com/trusty/amd64/libpython2.7-dev/filelist).

rbspy by contrast uses a bunch of header files from inside the Ruby interpreter, which makes it a
little more unstable and terrifying. This is because Python for whatever reason includes a lot more
struct definitions in its header files.

As a result, if you compare py-spy's [python bindings](https://github.com/benfred/py-spy/tree/master/src/python_bindings) to rbspy's [ruby bindings](https://github.com/rbspy/rbspy/tree/master/ruby-structs/src), you'll notice that

* there are way fewer Python binding files (6 vs 42 for Ruby)
* each file is much smaller (~30kb vs 200kb for Ruby)

Basically what I think this means is that py-spy is likely to be easier to maintain longterm than
rbspy -- since rbspy depends on unstable internal Ruby interfaces, future versions of Ruby could
break it at any time.

### the start of an ecosystem of profilers in Rust?? :)

One thing that I think is super nice is that rbspy & py-spy share some code! There's this
[proc-maps](https://github.com/benfred/proc-maps) crate that Ben extracted from rbspy and improved
substantially. I think this is awesome because if someone wants to make a py-spy/rbspy-like profiler
for another language like Perl or PHP or Javascript or something, it's even easier!

I have this secret dream that we could eventually have a suite of open source profilers for lots of
different programming languages that all have similar user interfaces. Today every single profiling
tool is different and it's a pain.

### also rbspy has windows support now!

Ben also contributed Windows support to [rbspy](https://github.com/rbspy/rbspy), which was amazing.
So if you want to profile Ruby or Python programs on Windows, you can!
