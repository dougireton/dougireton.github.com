---
layout: post
title: "Creating a git pre-commit hook for Chef cookbooks"
date: 2012-12-10 21:02
comments: true
categories: 
---

We've been using Chef in our group now for a few months, but until now we haven't been serious about linting or testing our Chef cookbooks. I decided to get serious today and write a Git pre-commit [hook][1] for linting cookboks.

Git runs the pre-commit hook script before each commit. This allows you to run code quality checks so only clean code is committed to your repo.

It's important to note that git hooks aren't copied down when you clone a git repo. Each developer will need to create his or her own pre-commit hook script in the .git/hooks/ directory of the repo. If you wanted to get fancy, you could keep git hook scripts in a "utility" repo and have a rake script to copy them to the right location.

The pre-commit script below does four things:

1. Runs a built-in Git whitespace check for trailing whitespace, mixed tabs and spaces, etc.
2. Runs ['knife cookbook test'] [2] to check Ruby and ERB template syntax.
3. Runs ['tailor'] [3] to check your code against Ruby style conventions.
4. Runs ['foodcritic'] [4], the de facto Chef cookbook linting tool.

[1]: http://git-scm.com/docs/githooks
[2]: http://wiki.opscode.com/display/chef/Managing+Cookbooks+With+Knife#ManagingCookbooksWithKnife-test 
[3]: https://github.com/turboladen/tailor
[4]: http://acrmp.github.com/foodcritic/

<!--more-->

{% include_code lang:ruby chef/pre-commit %}

## But, how do I use it?
Just copy the script below to file named 'pre-commit', make it executable, and copy it to the cookbooks/cookbook_name/.git/hooks/ directory.

## Wait a minute! It's not robust!
You may have noticed that the script needs a few things. It should check for the existence of various binaries (knife, foodcritic, tailor) before calling them. I'm sure you could think of many more improvents. I welcome your comments or gist forks. I just had to move on to more pressing things.

Thanks for reading and I welcome constructive comments...
