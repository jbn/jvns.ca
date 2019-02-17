---
title: "Linux comics: a small zine"
juliasections: ['Zines / comics']
date: 2017-11-25T09:56:47Z
url: /blog/2017/11/25/linux-comics--zine-edition/
categories: []
---

Last November, I drew a bunch of comics about Linux / computers / various systemsy things. One every
day, about 30 of them.  ([link 1](https://jvns.ca/blog/2016/11/27/more-linux-drawings/), [link 2](https://jvns.ca/blog/2016/11/10/a-few-drawings-about-linux/)). They're also at https://drawings.jvns.ca.

Since then, people have been asking me to write a book. I have not yet written a book. (one day!).
But!  This morning I woke up and thought, "well, what would happen if I just formatted all those
comics from last year into a small zine that folks could print?

So I put 24 comics together, ran my booklet-generating script, printed them, and voilà! A cute
little zine with 24 fun comics in it!

Flipping through it, I noticed a few factual errors here and there, so it's not perfect, but I
figure better to have it out in the world and imperfect than on my laptop where nobody can enjoy it.

### get the linux comics zine!

The zine is 24 comics like this:

<a href="https://jvns.ca/linux-comics-zine.pdf">
<img src="https://jvns.ca/images/linux-comics-cover.png" width=300px>
</a>

<a href="https://jvns.ca/linux-comics-zine.pdf">View it online</a>

<a href="https://jvns.ca/linux-comics-zine-print.pdf">Print version</a>

### "so you want to be a wizard" zine

In other zine news -- I also have a new zine called "so you want to be a wizard". It's about how
to learn hard things and get better at programming. It's [for sale on Gumroad](https://gumroad.com/products/TOOz) now if you want early access to it.

<img src="https://jvns.ca/images/so-you-want-to-be-a-wizard.png" width=300px>

### a script for formatting zines

I mentioned a "booklet-generating script" earlier in this post. What's that, you ask? I have a script I've been using to
format all of my zines into booklets. It's just 50 lines of Python (mostly argument parsing),
nothing fancy.

It took me quite a while to try to figure out how to wrangle all the PDF tools to do what I wanted,
so here it is: https://github.com/jvns/zine-formatter

One day I have to write a love letter to `pdftk` -- I've used pdftk to do so much pdf wrangling and
I do not know how I would make zines without it.
