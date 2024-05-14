---
layout: ../layouts/BlogPost.astro
title: Stop getting stuck in Vim and some other helpful git setup
slug: 2024-git-config-setup
description: Some helpful tips on getting your local git setup
tags:
  - technical
added: 2024-05-14T09:26:49.053Z
---

> Disclaimer: These are personal opinions/preferences carried out on a Mac you might have reasons for not using these same options, but I have found them helpful for my workflow.

There are a number of basic setup steps you can do to make your life easier when using git from the command line.

Inspired by a [video I watched this morning by Philomatics](https://www.youtube.com/watch?v=CAnQ4b0uais\&t=128s). I though I would expand on what has helped me out.

## Stop Getting Stuck In Vim

Some people love [Vim](https://www.vim.org/), but if like me you just don't like it or never got round to learning Vim, and you are forever forgetting that the command you need to use is `:wq!` to leave Vim any time you forgot to add a message on your `git commit` 🙈

Either [learn Vim](https://www.youtube.com/watch?v=-txKSRn0qeA).

Or save yourself some time and headache by changing your default editor to something else.

We are going to use [Visual Studio Code](https://code.visualstudio.com/) (vscode) for this but you can pretty easily find other options by searching `"git set text editor to <your editor of choice>"`.

First make sure you have installed the command line command for vscode.

1. Ensure Visual Studio Code Application is in your Applications folder.
2. Open vscode.
3. Open the command palette (__cmd + shift + P__).
4. Type `shell command`.
5. Click option to "Install 'code' command in PATH".
6. Start a new terminal or restart your terminal session for the new $PATH value to take effect.

If you have an error on step 5 try selecting the uninstall option and then try installing again.

You should now be able to start vscode from your command line 🎉

Example: `code .` can be used to open the current directory in vscode.

Now you can change the default editor to save you hours of wasted time quitting Vim.

In your terminal type:

```shell
git config --global core.editor "code --wait"
```

Now like with most of these changes in this post, take a second to pause and wonder why you didn't do this years ago and with all the time saved over the years whether you could have finished those 18 side projects 🤔

## Onto the Git config

[Full docs](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)

> ⚠️ I use the `--global` flag as I want these settings for all repositiories on my machine, you may want to have multiple gitconfigs with different setups for each repositiory.

We can now use `git config --global --edit` and it nicely opens in vscode or your chosen editor 🤯

This file might be looking pretty blank or it might have a bunch of stuff in already.

For example if you followed the git [getting started guide](https://git-scm.com/book/en/v2/ch00/ch01-getting-started) or had enough of git reminding you then you may have set up your name and email already:

```gitconfig
[user]
	name = Mark Hardwick
	email = mark@designibl.com
```

If you don't have the above in your config then you can simply paste it in and edit it to match the details of the git account you are using. Or use the following commands.

```shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

### Remote branch tracking/creation

If like me, you often start your workflow by making a branch locally with:

```shell
git checkout -b "mybranch"
```

Then upon trying to push or pull the branch git complains about no tracking infromation for the branches

> There is no tracking information for the current branch.
>
> Please specify which branch you want to merge with.
>
> See git-pull(1) for details
>
> git pull remote branch
>
> If you wish to set tracking information for this branch you can do so with:
>
> git branch --set-upstream-to=origin/branch mybranch

So then you re-run the command with the flag `--set-upstream-to=origin/mybranch`

Instead just setup the following git config:

```gitconfig
[push]
	autoSetupRemote = true
```

or run:

```shell
git config --global push.autoSetupRemote true
```

### Aliases for days

A note on aliases, I love them, they can save so much time however it is important to note 2 things.

1. If you are going to use aliases, understand what the commands do and know them so that you can help others without the same aliases setup. You will become dependant on them and have to re-set them up on every new machine.
2. Be careful that the alias you created doesn't use the same wording as another command as this will cause issues. Also make sure it is actually quicker to type than the original!

Onto the fun bit!

For any alias you can add it via the command line with:

```shell
git config --global alias.<alias> "<command>"
```

e.g. `git config --global alias.st "status -s"` is an alias for a shortened output of the git status command.

In the config file you will now have:

```gitconfig
[alias]
  st = status -s
```

So you can either keep adding to the list in the config or use the command line.

Aliases can be great for if you want to keep using the normal commands but often miss type them e.g. `git config --global alias.stts "status"`

#### Open config

```gitconfig
conf = config --global --edit
```

#### New local branch

```gitconfig
newb = checkout -b
```

#### Delete local branch

`-d` only deletes the branch if it has already been fully merged in its upstream branch.

You can changes this to `-D` which deletes the branch irrespective of its merged status.

```gitconfig
delb = branch -d
```

#### Commit with message

`-m` lets you write the commit message

```gitconfig
cm = commit -m
```

#### Edit message

```gitconfig
edit = commit --amend
```

> ⚠️ N.B. Amending modifies the commit history, to make things easy on everyone never amend pushed commits.

#### Update commit

`-a` adds all current changes

`--no-edit` prevents from edit the commit message.

```gitconfig
up = commit -a -ammend --no-edit
```

#### Update commit and edit the message

```gitconfig
upm = commit -a -ammend
```

#### Commit all unstaged changes with message

`-a` adds all unstaged changes to commit

```gitconfig
cma = commit -a -m
```

#### Unstage changes

```gitconfig
rh = reset HEAD
```

[Good video explanation by Philomatics](https://youtu.be/CAnQ4b0uais?t=255)

#### List all branches

`-a` flag is used to show the branches on origin too.

```gitconfig
lba = branch -a
```

#### Prune old branches

In my workflow I always delete the branch after every merged pull request so that our repository stays neat and I don't have a massive dropdown list of branches when making pull requests. But locally things aren't so neat. So you can use the following command to sync your local branches to match the origin branches.

N.B. This is a desturctive action so make sure you undertsand this command before using so you don't lose work.

```gitconfig
syncb = fetch -prune
```

### Even More Complex Aliases

When you get into the world of aliases you may find you want to be come even more efficient and run multiple commands at once etc.

But aliases have their limitations:

* Normal aliases can’t have parameters.
* You can’t execute multiple git commands in a single alias.
* You can’t use | (pipes) or grep.

Unless what if you could access the normal shell from an alias?

Introducing the !(bang) operator which allows us to do just this!

Now we can wrap our git command in an anonymous function.

```gitconfig
my_alias = "!f() { <your complex commands> }; f"
```

Now we can chain commands with `&&` 🎉

This also means we can pass in parameters with:

* $1 to mean the first parameter passed to the command, $2 to mean the second parameter (and so on for $3, $4, etc.) passed to the command.
* $@ to mean all command line parameters passed.

#### Grab all changes

```gitconfig
grab = "!git fetch && git pull"
```

#### Cleanup all merged branches

```gitconfig
bclean = "!f() { DEFAULT=$(git default); git branch --merged ${1-$DEFAULT} | grep -v " ${1-$DEFAULT}$" | xargs git branch -d; }; f"
```

[Credit Haacked](https://haacked.com/archive/2014/07/28/github-flow-aliases/) who provided this & has a great breakdown of how this works aswell.

So every so often I run `git bclean dev` and it removes every local branch I have already meregd into dev.

## Conclusion

Aliases can be super helpful when used correctly and you can create some complex workflows. But make sure you understand what they are doing, and that whichever way you use git, that you are aligned with your team.
