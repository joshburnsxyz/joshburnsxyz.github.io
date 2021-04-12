---
layout: post
title: A Spec for Managing Dotfiles
subtitle: 
date: "2021-04-12"
---

This blog post will serve as a rough draft of a specification document for managing dotfiles under git.
My primary goal is to find a way to managing linux configuration easily as possible and effectivly as possible.
Later on I will formalize a final copy and release it on my website as a follow up post.

## Concepts

### Filesystem Overlays
A __Filesystem Overlay__ refers to a folder tree that represents the linux filesystem hierachy. This used
to denote where files should be placed. for example if you have your `.bashrc` you would store it in a folder
structure such as `/home/me/.bashrc`. This is the basic structure of the entire spec.

### Install Scripts
__Install Scripts__ is exactly what it says. A script that automates the installation of the dotfiles.
It is reccomended this be implemented as a POSIX shell script. To prevent any need for dependencies on
the target system.

### External Configs
__External Configs__ refer to things like Emacs configurations, Fancy prompts like spaceship, things that 
exist in a seperate repo, usually these would be third-party, but not always.

## Example Dotfiles Repo
Here is an example of what this specification describes.

```
+ install.sh                   # Install script
+ overlay
    + home/
        + josh/                # Represents /home/<username>/
            + .bashrc
            + .bash_prompt
            + .xinitrc
            + .gitconfig
            + .config/
                + bspwm/
                + polybar/
                + sxhkd/
                + dunst/
                + termite/
```

And here is an example of that install script

```sh

#!/bin/sh

HOME_DIR="./overlay/home/josh"
TARGET_HOME="/home/josh"

fileList=".bashrc .bash_prompt .xinitrc .gitconfig .config"

echo "$fileList" | tr ' ' '\n' | while read item; do
  echo "Installing $item"
  cp -R "$HOME_DIR/$item" "$TARGET_HOME"
done

```
