---
layout: post
title: "Layout your .vimrc like a Boss"
date: 2013-02-23 15:21
comments: true
categories: vim, howto
---
I've been using [Vim][1] for a few years now starting with the excellent [Janus][2] distribution of Vim settings and plugins. As excellent as Janus is though, I really wanted to know each and every setting and plugin for myself. I didn't want any settings I didn't understand.

[1]: http://www.vim.org/
[2]: https://github.com/carlhuda/janus

About eight months ago I decided to get serious. I stripped down my .vimrc to about 20 settings I understood and could start using. I put my .vimrc and plugins in a [Git repo][3]. I slowly added settings one by one as I learned more about Vim.

[3]: https://github.com/dougireton/vimfiles

If you want to take a similar path, what follows is my recommended way of organizing your .vimrc to keep things organized.

<!--more-->

## Knobs and Dials

Vim has hundreds if not thousands of settings. It can be overwhelming or exciting, depending on your tolerance for fiddling with "knobs and dials". For a long time, I was frustrated with most .vimrcs I saw, including my own. They seemed like giant junk drawers of settings.

## The `:options` command
Finally, I hit upon a good organizing pattern. If you type `:options` in Vim's command mode, you get a giant list of Vim options grouped into related categories, e.g. "moving around, searching and patterns", "tags", "displaying text" and so on. These categories make an excellent way of grouping your own .vimrc settings. To get started, I just ran the `:options` command and copied each section heading in the same order as in the *option_window*.

```text
 1 important
 2 moving around, searching and patterns
 3 tags
 4 displaying text
 5 syntax, highlighting and spelling
 6 multiple windows
 7 multiple tab pages
 8 terminal
 9 using the mouse
10 printing
11 messages and info
12 selecting text
13 editing text
14 tabs and indenting
15 folding
16 diff mode
17 mapping
18 reading and writing files
19 the swap file
20 command line editing
21 executing external commands
22 running make and jumping to errors
23 language specific
24 multi-byte characters
25 various
```

## A snippet from my .vimrc

```vim

" ----------------------------------------------------------------------------
"  moving around, searching and patterns
" ----------------------------------------------------------------------------
set nostartofline     " keep cursor in same column for long-range motion cmds
set incsearch	      " Highlight pattern matches as you type
set ignorecase	      " ignore case when using a search pattern
set smartcase	      " override 'ignorecase' when pattern
                      " has upper case character

" ----------------------------------------------------------------------------
"  tags
" ----------------------------------------------------------------------------

" ----------------------------------------------------------------------------
"  displaying text
" ----------------------------------------------------------------------------
set scrolloff=3       " number of screen lines to show around
                      " the cursor
```

Once I had this organizing pattern I slowly added settings one by one, either by reading through the various Vim [options][5], or by reading through various .vimrcs I found online. I also picked up several great settings by reading [Practical Vim][6] by [Drew Neil][7]. 

Each time I added a setting, I made sure I understood it and added a explanitory comment to my .vimrc.

[5]: http://vimdoc.sourceforge.net/htmldoc/options.html
[6]: http://pragprog.com/book/dnvim/practical-vim
[7]: http://drewneil.com/

## Next Steps

If you don't already have a pattern for your .vimrc, I invite you to try my idea. I've found it makes my .vimrc more readable and sensible.

I also invite you to check out my [.vimrc and other Vim settings on Github][3]. I've tried to comment each setting in plain English so I know what the setting is when I'm reading it six months later. 

I'd love to hear any .vimrc related tips you have in the comments.
