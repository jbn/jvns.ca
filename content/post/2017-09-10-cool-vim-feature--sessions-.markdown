---
title: "Cool vim feature: sessions!"
juliasections: ['Cool computer tools / features / ideas']
date: 2017-09-10T12:58:53Z
url: /blog/2017/09/10/vim-sessions/
categories: ['vim']
---

Yesterday I learned about an awesome vim feature while working on my
[vimrc](https://github.com/jvns/vimconfig/blob/master/vimrc)! (to add fzf & ripgrep search plugins
mainly). It's a builtin feature, no fancy plugins needed.

So I drew a comic about it.

Basically you can save all your open files and current state with

```
:mksession ~/.vim/sessions/foo.vim
```

and then later restore it with either `:source ~/.vim/sessions/foo.vim` or `vim -S ~/.vim/sessions/foo.vim`. Super cool!

Some vim plugins that add extra features to vim sessions:

* https://github.com/tpope/vim-obsession
* https://github.com/mhinz/vim-startify
* https://github.com/xolox/vim-session

Here's the comic:

<div align="center">
<img src="https://jvns.ca/images/vimsessions.png">
</div>
