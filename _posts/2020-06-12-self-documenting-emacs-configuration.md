---
layout: post
title: Self Documenting Emacs Config
subtitle: With org-mode and use-package
date: "2020-06-12"
---

In this post I plan on going over how I built what I call
a *self-documenting* config. The reason it's called this
is because at its core, the config file is just a plain
text file. `org-mode` is one of the most powerful emacs
features and this is just more proof why.

## Loading an Org file from init

When emacs starts up, it looks for
a file `~/.emacs.d/init.el`. Normally
this is where you write your configuration.
We cant get past this fact, what we can
do use `init.el` to load the org-mode
file and build the config. We use a package
called `org-babel-load-file` that will
load the org file that will be our **actual**
configuration.

We will setup our loader like this
```lisp

(require 'org)
(org-babel-load-file (expand-file-name "~/.emacs.d/config.org"))

```

thats all we need to do now we will move over to our org file.

## How to configure emacs from org

We will use code blocks to write elisp code inside of our org 
file like this.

```org
#+BEGIN_SRC emacs-lisp
;;elisp code here
#+END_SRC
```

Everything between the blocks will be loaded as standard elisp code.
the rest is ignored. We can leverage this, and build a configuration
system that doesn't require a readme. You can seperate blocks under
headings and keep your configuration organised as-well. For further
reference check out my emacs config ~here~ *link redacted*.

