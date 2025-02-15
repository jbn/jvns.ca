---
title: "Prototyping an ltrace clone using eBPF"
juliasections: ['rbspy']
date: 2018-02-24T15:36:12Z
url: /blog/2018/02/24/an-ltrace-clone-using-ebpf/
categories: []
---

A few weeks ago on twitter, Leandro Pereira [suggested](https://twitter.com/lafp/status/960565232790732800) building a clone of ltrace using eBPF + bcc.

Yesterday and today I hacked together a prototype of an ltrace clone using eBPF + bcc! It's on
GitHub  and it's called [ltrace-bcc](https://github.com/jvns/ltrace-bcc). I think it's pretty cool
so here's a post about how it works!  This prototype uses the bcc crate for
Rust (https://github.com/jvns/rust-bcc) that I started building a few weeks ago.

The code is all in https://github.com/jvns/ltrace-bcc/blob/master/src/main.rs, it's like 200 lines
and I thought it was cool that it was possible to implement such a cool thing in so little code so I
wanted to explain how it works!

### what's ltrace?

ltrace is a program that traces library calls. You run `ltrace your-command`, and ltrace will tell
you what C functions (for example from libc) that command called! Here's an example of running
ltrace on `ls` and seeing it get a bunch of environment variables.

```
$ ltrace ls
...
getenv("QUOTING_STYLE")                          = nil
getenv("COLUMNS")                                = nil
ioctl(1, 21523, 0x7ffcfffdc7d0)                  = -1
getenv("TABSIZE")                                = nil
getopt_long(1, 0x7ffcfffdcc38, "abcdfghiklmnopqrstuvw:xABCDFGHI:"..., 0x414dc0, -1) = -1
getenv("LS_BLOCK_SIZE")                          = nil
getenv("BLOCK_SIZE")                             = nil
getenv("BLOCKSIZE")                              = nil
getenv("POSIXLY_CORRECT")                        = nil
getenv("BLOCK_SIZE")                             = nil
```

This is neat because it's a way to spy on what a program is doing that's different from spying on
its system calls!

### problems with ltrace

The main problem I have with ltrace is that even though there's a `-p` option ("Attach  to the
process with the process ID pid and begin tracing"), I don't think I've ever been able to get that
option to work. When I run `sudo ltrace -p SOME_PID`, nothing happens, even though I'm pretty sure
the process I'm tracing is calling library functions.

I don't fully understand **why** ltrace can't attach to processes, but that's not what this post is
about.

The other problem is that it introduces a lot of performance overhead (for the same reason that
strace does: it works using ptrace)! If I run the same program with and without ltrace, the ltraced
version will be way slower.

### goal: write a clone of ltrace that can attach to a running process

So I wanted to try to write an ltrace clone that could attach to a running process! I figured that I
could
prototype this pretty easily with the [rust bpf library](https://github.com/jvns/rust-bcc) I worked
on a few weeks ago.

Using eBPF + bcc to do tracing is in general faster than using ptrace to do tracing (which is what
ltrace does) so this approach should also be lower overhead and maybe more suitable for production
use.

### how does ltrace work? 

If we're going to try to write an ltrace clone, we need to understand how ltrace works at least a
little! The packagecloud blog has (among a lot of other great systems posts), a good post called [How does ltrace work?](https://blog.packagecloud.io/eng/2016/03/14/how-does-ltrace-work/) that explains. I won't rehash that whole post here, but here are the key points:

1. ltrace only traces dynamically linked functions (functions that are run from a dynamically loaded
   library, like libc or something)
1. to get a list of which functions to trace, ltrace parses the process's ELF file (looking at the
   PLT "Procedure Linkage Table")
1. ltrace inserts a breakpoint for each of those functions

We're not going to insert a breakpoint, so we can ignore everything about that. But we **do** need
to get a list of which functions to trace!

### getting a list of dynamically linked functions in a program

How can we find out which functions are dynamically linked into a program? That sounds hard.

It turns out it's not hard!!!!!!!! Basically if we run `readelf -a` we can find a bunch of function
names in the sections `rela.dyn` and `rela.plt`, and that lists the functions. It's not clear to me
what the difference between those two are yet. Some binaries seem to not have a `.rela.plt` section.

In Rust, this is literally 10 lines of code (+ comments :)). Here's some code that takes a filename
and prints out all the dynamically linked function names:

```
fn print_dynamic_functions(filename: &str) -> Result<(), Error> {
    // Read the file
    let mut contents: Vec<u8> = vec![];
    let mut elf = File::open(filename)?;
    elf.read_to_end(&mut contents)?;

    // parse the ELF using the goblin crate
    let binary = goblin::elf::Elf::parse(&contents).context("failed to parse ELF file")?;
    // load the function names from the `.rela.dyn` section
    for dynrel in binary.dynrelas {
        // we need to look up the symbol name (function name) in `binary.dynsyms` and
        // `binary.dynstrtab`.
        let sym = binary.dynsyms.get(dynrel.r_sym).unwrap();
        let name = binary.dynstrtab.get(sym.st_name).unwrap()?;
        println!("{:?}", name);
    }
    Ok(())
}
```

### using eBPF to trace functions

Okay, so once we have a list of functions that we want to trace, how do we actually trace them?
The way eBPF + uprobes work is -- you ask the Linux kernel to attach some eBPF code (that you write)
to a userspace function (for example `strlen` or something), and then that code can send some data
back to userspace.

Basically I wrote a super simple template that just saves  name into a struct and then sends the
event, and then replaced `NAME` with the name of every function I want to trace. So we generate
`trace_fun_strlen`, `trace_fun_strcpy`, etc.

```
let template = "
    int trace_fun_NAME(struct pt_regs *ctx) {
    struct data_t data = {};
    strcpy(data.function_name, \"NAME\");
    events.perf_submit(ctx, &data, sizeof(data));
    return 0;
};";
for name in &functions {
    bpf_code += &template.replace("NAME", &name);
}
```

Once we have a `trace_fun_strlen` function (which will submit an event every time it's called), we
need to attach it to the `strlen` function in libc. That's super easy:

```
for name in &functions {
    let uprobe_name = &format!("trace_fun_{}", name);
    let uprobe_code = module.load_uprobe(uprobe_name)?;
    module.attach_uprobe(library, name, uprobe_code, pid);;
}
```

### getting the data back from eBPF

Getting the messages back about which functions were called is super easy -- we just need to set up
a callback to print them out and then install the callback. here's what the code looks like in Rust:

```
int main {
    /* initial setup here */
    let table = module.table("events");
    let mut perf_map = init_perf_map(table, perf_data_callback)?;
    loop {
        perf_map.poll(200);
    }
}

fn perf_data_callback() -> Box<FnMut(&[u8]) + Send> {
    Box::new(|x| {
        let data = parse_struct(x);
        println!("{}({} [{}],{},{})", get_string(&data.function_name), data.arg1, get_string(&data.arg1_contents), data.arg2, data.arg3);
    })
}
```

### it works!!

Here are a couple of examples of using this ltrace prototype on a running Firefox process to figure
out what it's doing. I traced its calls to the pthread library and its calls to `libc`. I think it's
cool that we can see it locking and unlocking mutexes -- like you can see that each
`pthread_mutex_lock`  is followed with a corresponding `pthread_muted_unlock` on the same address.
For the calls to `strlen`, I made it read the string at the address of the first argument, so you
can see what string it's calculating the length of.

```
$ sudo ./target/debug/ltrace-bcc 16173 
Possible libraries:
/lib/x86_64-linux-gnu/libpthread.so.0
/lib/x86_64-linux-gnu/libdl.so.2
/usr/lib/x86_64-linux-gnu/libstdc++.so.6
/lib/x86_64-linux-gnu/libm.so.6
/lib/x86_64-linux-gnu/libgcc_s.so.1
/lib/x86_64-linux-gnu/libc.so.6
```

```
$ sudo ./target/debug/ltrace-bcc 16173 /lib/x86_64-linux-gnu/libpthread.so.0
pthread_mutex_unlock(140089540628720 [],0,140088461202304)
pthread_mutex_lock(140089913674096 [],140737445204276,0)
pthread_mutex_lock(140089539698712 [],0,0)
__errno_location(0 [],140737445203776,4294967295)
pthread_mutex_unlock(140089539698712 [],0,0)
pthread_mutex_lock(140089539698712 [],176,140737445204088)
pthread_mutex_unlock(140089539698712 [],176,140737445204088)
pthread_mutex_unlock(140089913674096 [],0,140089539698712)

$ sudo ./target/debug/ltrace-bcc 16173 /lib/x86_64-linux-gnu/libc.so.
clock_gettime(1 [],140737445202992,140088477396416)
strlen(140089705923984 [@mozilla.org/docshel],1,140089939581568)
strlen(140088631952416 [moz-extension://b6ba],140088631952384,4294967295)
strlen(140088469545224 [moz-extension://b6ba],140737445188512,140737445188504)
clock_gettime(1 [],140737445188400,0)
```

### writing debugging tools in Rust is fun

I think this is a cool example of why I'm excited about Rust -- like I got this idea to try to work
on an ltrace clone, and in a couple of days I had a prototype of something that kinda works!

It makes me happy that libraries to do things like parse ELF (goblin!!) are really easily available
in Rust! The definition of the [ELF struct from the goblin crate](https://docs.rs/goblin/0.0.14/goblin/elf/struct.Elf.html) to me
is a thing of beauty -- it just has everything I might to know about an ELF file in it!

Previously I was using the elf crate to parse ELF files but the goblin crate is WAY better, it has
support for parsing things like relocations (the `.rela.dyn` section) which the `elf` crate didn't.

I like that Rust crates generally don't try to make things simpler or harder than they are -- like
they don't introduce unnecessary abstractions, they just make it relatively easy for you get the
information you need.
