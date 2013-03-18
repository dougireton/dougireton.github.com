---
layout: post
title: "Flip your unicorn!"
date: 2013-03-07 20:47
comments: true
categories: vim
---

Let's say you had a unicorn in your code. Maybe something like this:

<pre>
                                           ________
                                        .##@@&&&@@##.
     \                               ,##@&::%&&%%::&@##.
     ^\^                            #@&:%%000000000%%:&@#
     /.(((                        #@&:%00'         '00%:&@#
    (,/"(((__,--.                #@&:%0'             '0%:&@#
        \  ) _( /{              #@&:%0                 0%:&@#
        !|| " :||              #@&:%0                   0%:&@#
        !||   :||              #@&:%0                   0%:&@#
        '''   '''              "" ' "                   " ' ""
</pre>

## But it's so wrong!
Clearly, something is wrong with the unicorn. It's not facing the rainbow. This week, we'll learn how to flip the unicorn (or any other text) with an awesome Vim [Visual mode mapping][2] courtesy of the inimitable [Dr. Chip][1]

[1]: http://www.drchip.org/astronaut/vim/index.html
[2]: http://www.drchip.org/astronaut/vim/index.html#Maps

<!--more-->

## Show me the map!

Here it is, just don't ask me to explain it.

```vim
vno  <silent> <Leader>fR   c<C-O>:set ri lz<cr><C-R>"<esc>:norm! dd`<<cr>:set ri! lz!<cr>
```

## And, here are the results\*

<pre>
            /       \               
          ^/^       ^\^          
        ))).\       /.(((        
 .--,__)))"\,)     (,/"(((__,--. 
}\ )_ (  /             \  ) _( /{
 ||: " ||!             !|| " :|| 
 ||:   ||!             !||   :|| 
 '''   '''             '''   ''' 
</pre>

Well, not exactly. The mapping doesn't exactly flip text. It won't turn a `)` into a `(`, or a `}` into a `{`. It just exchanges or swaps the places of the characters. Still it gets pretty close, much easier than trying to do it by hand. You can then run substitute commands to flip the `)`, `\`, and `}`.

## The Actual Results

<pre>
            \ 
          ^\^ 
        (((./ 
 .--,__((("/,(
{/ (_ )  \    
 ||: " ||!    
 ||:   ||!    
 '''   '''    
</pre>

## Uhhh, ok, why would you want to do this?

I have no idea, unless you had a unicorn you wanted to flip on a deadline. Thanks to Google and Dr. Chip for this (marginally useful) Vim tip! Until next week, cheers!

For the rest of the mappings which will flip text upside down as well, see [Dr. Chip's maps section][2] on his awesomely 1995 website.

<pre>
                                         ________
                                      .##@@&&&@@##.
                 /                 ,##@&::%&&%%::&@##.                   \                       
               ^/^                #@&:%%000000000%%:&@#                  ^\^          
             ))).\              #@&:%00'         '00%:&@#                /.(((        
      .--,__)))"\,)            #@&:%0'             '0%:&@#              (,/"(((__,--. 
     }\ )_ (  /               #@&:%0                 0%:&@#                 \  ) _( /{
      ||: " ||!              #@&:%0                   0%:&@#                !|| " :|| 
      ||:   ||!              #@&:%0                   0%:&@#                !||   :|| 
      '''   '''              "" ' "                   " ' ""                '''   ''' 
</pre>
