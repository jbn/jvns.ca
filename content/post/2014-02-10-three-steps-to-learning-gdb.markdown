---
categories: ["spytools"]
juliasections: ['How a computer thing works']
comments: true
date: 2014-02-10T00:00:00Z
title: Three steps to learning GDB
url: /blog/2014/02/10/three-steps-to-learning-gdb/
---

Debugging C programs used to scare me a lot. Then I was writing my
[operating system](http://jvns.ca/blog/categories/kernel) and I had so
many bugs to debug! I was extremely fortunate to be using the emulator
qemu, which lets me attach a debugger to my operating system. The
debugger is called `gdb`.

I'm going to explain a couple of small things you can do with `gdb`,
because I found it really confusing to get started. We're going to set
a breakpoint and examine some memory in a tiny program.

<!--more-->

### 1. Set breakpoints

If you've ever used a debugger before, you've probably set a
breakpoint.

Here's the program that we're going to be "debugging" (though there
aren't any bugs):

~~~
#include <stdio.h>
void do_thing() {
    printf("Hi!\n");
}
int main() {
    do_thing();
}
~~~

Save this as `hello.c`. We can debug it with gdb like this:

<pre>
bork@kiwi ~> gcc -g hello.c -o hello
bork@kiwi ~> cat
bork@kiwi ~> gdb ./hello
</pre>

This compiles `hello.c` with debugging symbols (so that gdb can do
better work), and gives us kind of scary prompt that just says 

`(gdb) `

We can then set a breakpoint using the `break` command, and then `run`
the program.

<pre>
(gdb) break do_thing 
Breakpoint 1 at 0x4004f8
(gdb) run
Starting program: /home/bork/hello 

Breakpoint 1, 0x00000000004004f8 in do_thing ()
</pre>

This stops the program at the beginning of `do_thing`.

We can find out where we are in the call stack with `where`: (thanks
to [@mgedmin](https://twitter.com/mgedmin) for the tip)

<pre>
(gdb) where
#0  do_thing () at hello.c:3
#1  0x08050cdb in main () at hello.c:6
(gdb) 
</pre>

### 2. Look at some assembly code

We can look at the assembly code for our function using the
`disassemble` command! This is cool. This is x86 assembly. I don't
understand it very well, but the line that says `callq` is what does
the `printf` function call.

<pre>
(gdb) disassemble do_thing
Dump of assembler code for function do_thing:
   0x00000000004004f4 <+0>:     push   %rbp
   0x00000000004004f5 <+1>:     mov    %rsp,%rbp
=> 0x00000000004004f8 <+4>:     mov    $0x40060c,%edi
   0x00000000004004fd <+9>:     callq  0x4003f0 <puts@plt>
   0x0000000000400502 <+14>:    pop    %rbp
   0x0000000000400503 <+15>:    retq 
</pre>

You can also shorten `disassemble` to `disas`

### 3. Examine some memory!

The main thing I used `gdb` for when I was debugging my kernel was to
examine regions of memory to make sure they were what I thought they
were. The command for examining memory is `examine`, or `x` for short.
We're going to use `x`.

From looking at that assembly above, it seems like `0x40060c` might be
the address of the string we're printing. Let's check!

<pre>
(gdb) x/s 0x40060c
0x40060c:        "Hi!"
</pre>

It is! Neat! Look at that. The `/s` part of `x/s` means "show it to me
like it's a string". I could also have said "show me 10 characters"
like this:

<pre>
(gdb) x/10c 0x40060c
0x40060c:       72 'H'  105 'i' 33 '!'  0 '\000'        1 '\001'        27 '\033'       3 '\003'        59 ';'
0x400614:       52 '4'  0 '\000'
</pre>

You can see that the first four characters are 'H', 'i', and '!', and
'\0' and then after that there's more unrelated stuff.

I know that gdb does lots of other stuff, but I still don't know it
very well and `x` and `break` got me pretty far. You can read the
[documentation for examining memory](http://ftp.gnu.org/old-gnu/Manuals/gdb-5.1.1/html_chapter/gdb_9.html#SEC56).
