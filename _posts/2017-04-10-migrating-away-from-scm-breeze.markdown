---
title: Migrating away from SCM Breeze
layout: post
categories:
    - blog
    - tech
---

I had been using [SCM Breeze](https://github.com/scmbreeze/scm_breeze) for a
few years and while it's a great project with many ways to configure it, it
also created a lot of frustration for me because of bugs and other problems it
caused. I had a look at the code which seemed rather complex, especially after
I understood what it did, and I figured I would give writing me own version a
shot.

## What is SCM Breeze doing anyway?

To me, the main feature is the one adding indexes to `ls`, `git status` and
`git branch` output so I do not have to type or copy-paste file names, but can
instead use the indexes in other commands. It also adds loads of aliases and
has a feature called `Repository Index` of which I only used quick access with
tab completion to the folder with my git clones. Any other features it might
have, I did not use. The same for most of the aliases coming with SCM Breeze -
I tend not to use things if I did not configure them myself.  So as a recap, I
needed the following:

 - aliases to save typing
 - a command for quick access to my git clones
 - a way to add indexes to command output
 - a way to use these indexes in other commands

## My goals

 - simple code
 - no hijacking of commands (SCM Breeze wrapped git commands and the commands
   using the indexes which is not very flexible and caused problems with
   completions for me)
 - keep the command output as close as possible to the original (instead of
   parsing `--porcelain` output and creating my own like SCM Breeze does)
 - ability to use the indexes whereever I wanted without extra configuration
 - no need for rebuilding a repository index (which I regularly forgot so I
   could not access new clones the usual way)
 - even though I am a ZSH user, the code should be as reusable under bash as
   possible.

## The solution

The simplest part were the aliases, I could simply configure everything I
needed using ZSH - it would be the same with bash. The quick access function I
wrote uses ZSH specific functionality, but can surely be easily implemented in
bash to:

```bash
function c() {
    cd ~/Clones/$1;
}
# complete with ~/Clones prefix
compctl -/ -W ~/Clones/ c

```
Now I can simply type `c rep<Tab>` and I will end up in `~/Clones/repository`.

With the simple stuff out of my way, I could go about the index feature. I
wanted to stay compatible with the variables SCM Breeze uses, so I started with
the index consumer part. SCM Breeze creates numbered environment variables to
hold references to file paths and branches, so all that needed to be done was
parsing indexes from the command line, resolve the matching environment
variable and replace the indexes with the result. What I came up with is the
[expand-indexes-or-expand-or-complete function][1] which I map to `<Tab>` like
[this][2] Now I can type `comamnd 1 2 3-6 7<Tab>` and the numbers will be
replaced with branches or file paths that are referenced by the environment
variables matching the numbers / range of numbers (normal tab completion still
works if there are no numbers or matches). Now obviously this is not the same
as just typing the number and enter and the command will just run, but I use
`<Tab>` all the time anyway and this approach has the following advantages:

 - I can use indexes with all commands
 - I can double check that I typed the right numbers
 - I can expand the same number twice and edit the result for simple renaming
 - shell history search has more of a chance to find something useful when
   having the history full of `vim meaningful_filename` instead of `vim 1`

Not sure whether it would work under bash but it could as I am pretty sure I
have already seen something like global bash completions. Command wrapping
approach is always possible if not.

Off to the last part: adding indexes to command output. I rarely use full
commands but mostly aliases, so I figured I can create aliases where I call
scripts/functions to do the job. So I created a [Python script][3] and added
it to my `PATH` that adds the indexes and outputs a list of files to be used in
a subsequent step. This list is filtered from the output and transformed into
environment variables by this simple [function][4] (it needs to be a function,
not a script as environment variables can only be set inside the same process,
which a call to a script would leave.) I use them in [these functions][5]. The
only reasone it's functions and not aliases is because I need to be able to
pass arguments to the first command in the pipeline.

Done!

[1]: https://github.com/sblask/dotfiles/blob/774d79c7595777725a864ce55cf6bacd41c37a77/zshrc.d.dotfile/index.zsh#L52
[2]: https://github.com/sblask/dotfiles/blob/774d79c7595777725a864ce55cf6bacd41c37a77/zshrc.d.dotfile/keybindings.zsh#L55
[3]: https://github.com/sblask/dotfiles/blob/774d79c7595777725a864ce55cf6bacd41c37a77/.bin/add-index.symlink
[4]: https://github.com/sblask/dotfiles/blob/774d79c7595777725a864ce55cf6bacd41c37a77/zshrc.d.dotfile/index.zsh#L1
[5]: https://github.com/sblask/dotfiles/blob/774d79c7595777725a864ce55cf6bacd41c37a77/zshrc.d.dotfile/index.zsh#L77
