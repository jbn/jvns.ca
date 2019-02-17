---
title: "What's a reference in Rust?"
juliasections: ['Rust']
date: 2017-11-27T23:07:00Z
url: /blog/2017/11/27/rust-ref/
categories: []
---

Hello! Recently I am trying to learn Rust (because I am going to do a project in Rust, and to do
that I need to learn Rust better). I've written a few hundred lines of Rust over the last 4 years,
but I'm honestly still pretty bad at Rust and so my goal is to learn enough that I don't get
confused while writing very simple programs.

The audience I'm writing for in this post is a little specific -- it's something like "people who
have read the [lifetimes chapter in the Rust book](https://doc.rust-lang.org/1.9.0/book/lifetimes.html) and sorta understand it in principle but are still confused about a lot of pretty basic Rust things."

we are going to talk about

* What even is a reference in Rust?
* What is a boxed pointer / string / vec and how do they relate to references?
* Why is my struct complaining about lifetime parameters and what should I do about it?

We are not going to talk about ownership / the borrow checker. If you want to know about ownership you should read [the rust book](https://doc.rust-lang.org/book/second-edition/ch04-01-what-is-ownership.html). Also there is probably at least one mistake in this post.

Let's start with something extremely basic: defining a struct in Rust.

```rust
struct Container {
    my_cool_pointer: &u8,
} 
```

Pretty simple, right? When I try to compile it, the compiler gives me this error message:

```
2 |     my_cool_pointer: &u8,
  |                      ^ expected lifetime parameter``
```

### Why doesn't this compile?

Digging into why this simple program doesn't compile is interesting and helped me understand
some things about Rust a tiny bit better.

There is a straightforward answer ("it's missing a lifetime parameter"), but that answer is not that
useful. So let's ask another better question instead!

### What does `&` mean?

I didn't fully appreciate what `&` meant in Rust until yesterday. A `&` is called a "reference" and it
is very interesting! You might be thinking "julia, this is boring, I KNOW what a reference is, it is
like a pointer in C, I know what that is".

BUT DO YOU REALLY??

Okay, so let's say you have a pointer/reference to some memory, `&my_cool_pointer`. That memory
could be in one of 3 places:

1. on the heap
2. on the stack
3. in the data segment of your program

The most important thing about Rust (and the thing that makes programming in Rust confusing) is that
it needs to decide **at compile time** when all the memory in the program needs to be freed.

So: let's say I've written a program like this.

```
fn blah () {
    let x: Container = whatever();
    return;
}
```

When the function `blah` returns, `x` goes out of scope, and we need to figure out what to do with
its `my_cool_pointer` member. But how can Rust know what **kind** of reference `my_cool_pointer` is?
Is it on the heap? Who knows??

So this is no good, Rust can't compile this program because it doesn't have enough information about
what kind of reference our reference is.

(to understand what references are it is also helpful to read the [rust book on references & borrowing](https://doc.rust-lang.org/book/second-edition/ch04-02-references-and-borrowing.html), that chapter says different things about references which are also true and useful)

### making our struct compile with a `Box`


If we **knew** that `my_cool_pointer` was allocated on the heap, then we would know what to do when
it goes out of scope: free it! The way to tell the Rust compiler that a pointer is allocated on the
heap is using a type called `Box`. So a `Box<u8>` is a pointer to a byte on the heap.

This code compiles!!

```
struct Container {
    my_cool_pointer: Box<u8>,
} 
```

We can now use our Container struct in a program and run it:


```
struct Container {
    bytes: Box<u8>,
} 

fn test() -> Container {
    // this is where we allocate a byte of memory
    return Container{bytes: Box::new(23)}; 
}

fn main() {
    let x = test();
    // when `main` exits, the memory we allocated gets freed!
}
```

### boxes in Rust vs boxes in Java

As an aside -- I got a bit confused by the word "box". I know in Java you have boxed pointer versions of
primitive types, like `Integer` instead of `int`. And you can't really have non-boxed pointers in Java,
basically every pointer is allocated on the heap.

Rust boxed pointers (`Box<T>`) are a bit different from Java boxed pointers though!

In Java, a boxed pointer includes an extra word of data (not sure what it's for exactly, but I know
there's an extra word for something).

In Rust, a boxed pointer **sometimes** includes an extra word (a "vtable pointer") and sometimes
don't. It depends on whether the `T` in `Box<T>` is a type or a trait. Don't ask me more, I do not
know more.

Anyway, when you have a boxed pointer, the compiler uses the information that it's allocated on the
heap in order to decide where in the compiled code to free the memory. In our example above, the
compiler would insert a `free` at the end of the `main` function.

### what if you want to point to existing memory?

Okay, so now we know how to allocate new memory and refer to it (use a boxed pointer!). But what if
you want to refer to some **existing** memory somewhere and point to that?

A good example of this is -- I've used this DWARF parser called
[gimli](https://github.com/gimli-rs/gimli). It doesn't do basically any allocations -- it just loads
all the DWARF data for your program into memory and then points into that data.

This is where you use lifetimes. I'm not going to explain lifetimes because they're [explained in the Rust book](https://doc.rust-lang.org/1.9.0/book/lifetimes.html)

This program also compiles:

```rust
struct Container<'a> {
    my_cool_pointer: &'a u8,
} 
```

So now we have made our struct compile in 2 different ways: by adding a lifetime parameter to our struct
definition, and by using a boxed pointer instead of a reference. Great!

Let's talk about things that are allocated on the heap a little more.

### How do you know if a Rust variable is allocated on the heap?

So, we learned in this post that if a variable has type `Box<T>`, then it's a pointer to some memory
on the heap. Are there other types that are always on the heap? It turns out there are!! 
Here they are:

* `Vec<T>` (an array on the heap)
* `String` (a string on the heap)
* `Box<T>` (just a pointer). 

These 3 types all have equivalent reference types (again: a reference is a pointer to memory in an unknown place):
`&[T]` for `Vec<T>`, `&str` for `String`, and `&T` for `Box<T>`.

I think these 3 types (`Vec<T>`, `String`, and `Box<T>`) are very important in Rust and undersanding
the relationship between them and their reference version (`&[T]`, `&str`, `&T`) is extremely
important when writing Rust programs. Like I didn't understand before and I think that has been part
of why I was so confused about Rust.

Converting from a `Vec<T>` to a `&[T]` is really easy -- you just run `vec.as_ref()`. The reason you
can do this conversion is that you're just "forgetting" that that variable is allocated on the heap
and saying "who cares, this is just a reference". `String` and `Box<T>` also has an `.as_ref()`
method that convert to the reference version of those types in the same way.

You can't as easily convert back from a `&[T]` to a `Vec<T>` though (because a `&[T]` could be
a pointer to memory on the stack, so it doesn't make sense to just say "this is something on the
heap")! To convert back, you'd need to make a clone and allocate new memory on the heap.

### there are 2 kinds of structs: those with lifetimes and those without lifetimes

So we have arrived at a useful fact about Rust!

Every struct (or at least every useful struct!) refers to data. Some structs have lifetimes as part
of their type, and some don't.

Here's another example of a struct with no lifetime:

```
struct NoLifetime {
    x: Vec<u8>,
    y: String,
    z: Box<u8>
}
```

This struct has pointers to an array, a string, and a byte on the heap. When an instance of this
struct goes out of scope all that memory will be freed.

Next, here's a struct with a lifetime! This struct also has pointers to an array, a string, and a
byte. We have no idea from this struct definition where the data this is pointing to will be.

```
struct Lifetime<'a> {
    x: &'a [u8],
    y: &'a str,
    z: &'a u8
}
```

I can't tell you which kind of struct to make your Rust structs because I don't know yet.

### do structs in Rust usually have lifetimes or not?

One question I have (that I think I will just resolve by getting more Rust experience!) is -- when I
write a Rust struct, how often will I be using lifetimes vs making the struct own all its own data?

I looked in https://github.com/BurntSushi/ripgrep, and most of the struct definitions there do not
have lifetimes. I don't know what that tells me!

### that's all for now

I learned a lot of useful things by writing this blog post! Now I am going to go back to writing
Rust programs and see if all this newfound knowledge helps me write them in a less confused way.
