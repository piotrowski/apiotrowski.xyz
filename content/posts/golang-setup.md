---
title: "Golang + Vim"
date: 2020-11-23
categories: ["Vim"]
---

Is Golang + Vim worth trying? Definitely, developing an application with this combination is a cool experience. However I tried using it on a bigger project (I was doing bugfix on docker-cli) and that wasn't a pleasing experience, I felt a bit lost and tired of all of these shortcuts and jumping between files, I think it my be a better experience after I get more experience with Vim. Below I am showing how to setup Vim for Go programming. It's my first attempt so it may be still lacking some features :D

<!--more-->

## Plugins

I have used two plugins to get everything I needed:
```bash
Plug 'preservim/nerdtree'
Plug 'fatih/vim-go', { 'do': ':GoUpdateBinaries' }
```

### NERDTree

NERDTree is a cool tool for every developer, as it allows you to create/move/delete files and directories from inside Vim and most importantly it allows you to browse files just like in any other editor. Whole experience was pretty good, I think some stuff is a bit unnatural for somebody who is coming from IDE word, buy you can get used to it. Also there is a lot of stuff that I was able to do which is not possible in VSCode e.g. split screen in both directions at once.

> If you want to split screen with multiple files, use `s` to split vertically and `i` to split horizontally.

Below you can see my simple configuration for it, as I wanted to make it more approachable:

```bash
"NERDtree
map <C-b> :NERDTreeToggle<CR>
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif
set autochdir
let NERDTreeShowHidden=1
```

With those I was able to use NERDTree as I like. E.g. with `Ctrl+b` I can open the tree file view.

> `set autchdir` was really important for me as i usually open vim like this: `vim ~/path/to/file` and NERDTree wouldn't catch up and root of NERDTree was still in my current location which was annoying.

### faith/vim-go

This one is really powerful, I was able to do everything that I was used to after using VSCode golang extension. I think it worked even better than VSCode E.g I always had some issues with importing my own packages, however here that was not a problem. Maybe it was something else, as it uses the same golang tools as VSCode but I thought it is good to mention.

Configuration I used:
```bash
"vim-go
let g:go_fmt_command = "goimports"
let g:go_auto_type_info = 1
```

> By default fmt tool uses a different command to format an output so I changed it, as goimports formats code too and does some additional stuff. If you want to read more, check out links at the end of article.

## Summary

With those two plugins my developer experience was quite good, but as I wrote at start, only for smaller applications. I feel like I have to get used to Vim a bit more before jumping further with it.

## Further reading

- Vim for blogging. This setup is great for anybody who like to write blog posts in markdown. I am a big fun of this setup. Read more [HERE](/posts/setup-for-bloggers/)
- Good tutorial with basic NerdTree functionality - [medium](https://medium.com/usevim/nerd-tree-guide-bb22c803dcd2)
- NERDTree github, best source of NERDTree knowledge - [link](https://github.com/preservim/nerdtree)
- Why I changed to goimports - [link to github issue](https://github.com/fatih/vim-go/issues/207)
