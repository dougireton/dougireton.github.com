---
layout: post
title: "Make Vim's Goto File command open included Chef recipes"
date: 2013-01-27 21:18
comments: true
categories: vim, chef, howto
---
Ohai Chefs!

Last week, we looked at how to get [Chef clients grouped by version][1]. This week, we'll look at something completely different - customizing Vim to jump to included Chef recipes via the [Goto File command][2] - `gf`.

[1]: http://dougireton.com/blog/2013/01/19/get-chef-clients-by-version/
[2]: http://vim.wikia.com/wiki/Open_file_under_cursor#Go_to_file

## TL;DR
Add this to your .vimrc to be able to jump to included recipes (via `include_recipe`). Caveat: you can only jump to fully-qualified recipe names, e.g. `my_cookbook::my_recipe`. Hitting `gf` on `include_recipe 'my_cookbook'`, won't jump to `my_cookbook::default`.

```vim .vimrc autocmd for include_recipe
" Make gf work on Chef include_recipe lines
" Add all cookbooks/*/recipe dirs to Vim's path variable
autocmd BufRead,BufNewFile */cookbooks/*/recipes/*.rb setlocal path+=recipes;/cookbooks/**1
```

Read on to find out why this works.
<!--more-->

## Including Recipes
In Chef you can [include one recipe in another recipe][3]. For example, in one of our "application" cookbooks our `default.rb` recipe contains about 15 `include_recipe` statements which include recipes from the cookbook as well as recipes from other cookbooks. Often, I want to open one of the included recipes from the _default_ recipe.

[3]: http://docs.opscode.com/essentials_cookbook_recipes_in_recipes.html

## Vim Paths

```vim .vimrc autocmd for include_recipe
" Make gf work on Chef include_recipe lines
" Add all cookbooks/*/recipe dirs to Vim's path variable
autocmd BufRead,BufNewFile */cookbooks/*/recipes/*.rb setlocal path+=recipes;/cookbooks/**1
```

The autocmd above runs when you read in an existing file (BufRead) or create a new file (BufNewFile) in a Chef cookbook `recipes` directory. The directory path `*/cookbooks/*/recipes/*.rb` assumes you have a `cookbooks` directory which contains your cookbooks and inside each cookbook you have a `recipes` directory.

The `setlocal` sets the path option locally for the recipe file, not globally in Vim.

The actual `path` value deserves a bit more explanation. `path+=recipes;/cookbooks/**1` means append `recipes` to the current working directory, and also go upwards to the `cookbooks` directory and search all directories one level down from `cookbooks` and append `recipes` to those as well. So in other words it will search for the file under the cursor in the current cookbook's `recipe` directory, and all other cookbooks' `recipe` directories as well.

See Vim's help on [file searching][4] for more info.

[4]: http://vimdoc.sourceforge.net/htmldoc/editing.html#file-searching

## Conclusion

So there you have it. 

1. Throw that line in your .vimrc file
2. ??
3. [Profit][5]

Being able to jump to included Chef recipes is a handy little trick, but there is one caveat. It doesn't work for default recipes, e.g. `include_recipe "foo"`. The next step for this trick is to use the `includeexpr` [option][6] to add `default.rb` to the filename.

Next week, we'll look at various knife search tricks for getting data from your Chef server.

[5]: http://www.youtube.com/watch?v=tO5sxLapAts
[6]: http://vimdoc.sourceforge.net/htmldoc/options.html#'includeexpr'

