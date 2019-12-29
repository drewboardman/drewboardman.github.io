---
layout: post
title:  "Setting up a Haskell environment in NeoVim"
date:   2019-12-29 11:01:02 -0400
categories: jekyll update
---
I've been spending a lot of time recently getting my Haskell development setup
as nice as possible. I found out a couple gotchas, so thought I'd write them
down here. I have this same setup working on Ubuntu 18.04 and MacOS Catalina.
However I'm writing this guide with linux in mind, it should be a small change
to get it to work on your distro (or with Mac).

![image](/assets/images/coc_setup.png)

`nvim`
-------
* For both `brew` and `apt`, you're going to be downloading a version of neovim
  that isn't compatible with the most useful of the LSP features we want. So
  you're going to install it from [a pre-built
  package](https://github.com/neovim/neovim/releases/). On linux, it's the
  *appimage*.
* Make sure you put this somewhere on your path. For me I have it in
  `~/bin/nvim` (I cut the `.appimage` from the binary, since I have `vim`
  aliased to `nvim`).
* For some cool colorschemes and such, you can check out [my
  dotfiles](https://github.com/drewboardman/dotfiles/tree/master/neovim/.config/nvim).

`nodejs`
---------
* You're going to need a newer version of node than `apt` will give you. The
  easiest way to do this is via `nvm`. It's very simple to install the latest
  version of node via [their installation
  instructions](https://github.com/nvm-sh/nvm).

`coc.nvim`
----------
* If you aren't using it already, I recommend [migrating to
  `vimplug`](https://github.com/junegunn/vim-plug). Either way, you're going to
  need the coc.nvim plugin. Since you already set up node from the previous
  section, you just need to follow their config instructions and you're good to
  go. [Instructions here](https://github.com/neoclide/coc.nvim).

`stack`
--------
* You're going to want either stack or cabal. I use stack, and [here is how you
  install it](https://docs.haskellstack.org/en/stable/README/).

`hie`
--------------------
* This is the part that was trickiest for me. All of the instructions are on
  [the
  README](https://github.com/haskell/haskell-ide-engine/blob/master/README.md),
  but they are pretty hard to follow with how many systems they try to account
  for.
  - First thing, you're going to need to clone the hie repo somewhere, and
    install the version for the project(s) you want to use it on. If you don't
    know which version of GHC you're on, and you have a stack project - just
    check out the `stack.yaml` file and google that resolver. I'm on resolver
    14.18, which is ghc version 8.6.5

    ```
    $ stack ./install.hs hie-8.6.5
    ```

  - This takes a while, so just let it run.

* Once this is done, you're going to need some configuration. `coc.nvim` has [a
  haskell
  config](https://github.com/neoclide/coc.nvim/wiki/Language-servers#haskell).
  However, you **need to understand this important point** with this config.
  What this assumes is that you have a binary `ghc` available on your path that
  matches the project ghc. This is almost certainly not (and shouldn't) be the
  case if you're using stack. You need to tell coc.nvim that you're using stack,
  and that you want stack to manage the `ghc` executable.
  - To do this, make sure you have stack's package directory on your path. So
    add it (the default is `~/.local/bin`) to your `.zshrc` or `.bash_profile`.
  - Now you need to configure `coc.nvim` to use this `ghc` with the following:

  ```json
  "languageserver": {
    "haskell": {
      "command": "stack",
      "args": ["exec", "hie-wrapper"]
    }
  }
```

Final Setup
------------
* Make sure you `stack build` in your project before you open it in nvim.


There you have it, a nice `nvim` experience with haskell. Some tips:

* Some useful keybinds that come by default:

  - `shift + k` inspects type/documentation
  - `g d` goes to a def

* I have some cool color fixes for the `coc.nvim` popup:

```
" Coc popup colors are bad, this makes them not bad
highlight CocFloating cterm=NONE guibg=DarkSlateGrey guifg=burlywood
highlight CocErrorFloat cterm=NONE guibg=DarkSlateGrey guifg=burlywood
highlight CocWarningFloat cterm=NONE guibg=DarkSlateGrey guifg=burlywood
highlight CocInfoFloat cterm=NONE guibg=DarkSlateGrey guifg=burlywood
```

Feel free to check out [my
dotfiles](https://github.com/drewboardman/dotfiles/tree/master/neovim) for some
more configs.
