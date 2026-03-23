---
title: "Vim setup for bloggers"
date: 2020-10-07
categories: ["Vim"]
---

In this article, I'll share some valuable tips to harness the power of Vim as a versatile tool for blog writing. From migrating to Neovim to incorporating spellchecking and useful shortcuts, these tips will streamline your writing process and boost productivity.

<!--more-->

### Embracing Neovim

Despite its age, Vim remains a robust text editor with a dedicated following. However, Neovim, a refreshed fork from 2014, offers several enhancements and modifications that make it a compelling choice for modern writers.

To get started with Neovim, you can easily install it via the command line using your distribution's package manager:
```bash
sudo apt-get install neovim
```

#### Basic configuration
Neovim's configuration files are located in the *~/.config/nvim/* directory. File *init.vim* is the main source for Vim configuration. Syntax is the same as in standard Vim, so you can use the same file for both versions of Vim.

```bash
set number
set ruler

set expandtab
set tabstop=4
set softtabstop=4
set shiftwidth=4
```
This configuration ensures that essential settings like line numbering and tab length are pre-configured, eliminating the need to adjust them every time you open the editor.

#### Plugin manager

Installing plugins in Neovim can be cumbersome without proper management tools. Fortunately, plugins like [vim-plug](https://github.com/junegunn/vim-plug) simplify this process. You can install it with a single command:

```bash
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
```

After installation, create a plugged directory in *~/.config/nvim/* and add plugin declarations to your *init.vim* file:
```bash
call plug#begin()
Plug 'ron89/thesaurus_query.vim'
Plug 'mzlogin/vim-markdown-toc'
call plug#end()
```
Following lines will inform *vim-plug* that you want to install two modules. Save and call `:PlugInstall` and add-ons will be installed.

### Spellchecker Integration

First thing I wanted inside Vim was a spellchecker so I will not do any stupid mistakes like e.g. Mistkae. Every misspelling will glow, when we navigate our cursor over that word, we can use `z=` shortcut to open a dictionary with plenty of propositions on how we can fix the mistake.

Another useful shortcuts are:
- `]s` and `[s`. They will move your cursor to next/prev mistake.
- `zw` `zuw` `zg` `zug` - add/remove word from wrong/good word list
If you want to learn more commands for this tool, just use `:help spell` to call manual.

Spell utility doesn't require any plugins as it is build into nvim; Just paste those three lines into *init.vim*
```bash
autocmd FileType markdown setlocal spell
set spelllang=en
hi SpellBad ctermfg=white ctermbg=red
```
First two lines are required for the spellchecker to work, last line was added to customize how mistakes look inside the editor. By default errors in my neovim were highlighted with pink/purple color. However I find the red highlight more readable.

### Thesaurus synonyms

After installing the plugin (Already done in Plugin Manager step) the only thing you need to know is the following shortcut:
```bash
<Leader>cs
```

> **INFO**:
> Leader is a key you can configure by yourself. By default <Leader> is a `\`(frontslash) key

### Markdown TOC

Just like previous section. Plugin is already installed if you followed Plugin Manager section. To create Tree Of Content that is automagically updating on each save, call command `:GenTocGFM`. For other useful commands check plugin github repository ([Link](https://github.com/mzlogin/vim-markdown-toc))

### Undo and redo inside interactive mode

This one is pretty interesting. In Vim you can customize everything and test it on the fly. E.g this configuration can be pasted into _init.vim_ to have it permanently or typed inside Vim to test only for this session:
```bash
"Mappings
:inoremap <C-z> <ESC>ui
:inoremap <C-r> <ESC><C-r>i
```

With this configuration, `Ctrl+z` in interactive mode will work like in any other modern editor. There are plenty of other options like for example, you can print current time with <F3> - `:nnoremap <F3> :echo system("date")<CR>`
There are plenty other configurations you can create. It's all up to what works best for you.

> **INFO:** Do not forget about that **i** at the end of mapping command because, without it you will leave interactive mode.
