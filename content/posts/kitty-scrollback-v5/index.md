+++
title = 'kitty-scrollback.nvim v5.0.0 released!'
date = 2024-06-07T09:06:11-04:00
publishDate = 2024-06-07T09:06:11-04:00
draft = false
tags = ['neovim', 'kitty', 'kitty-scrollback.nvim', 'tmux']
description = 'kitty-scrollback.nvim v5.0.0 release announcment'
summary = "kitty-scrollback.nvim v5.0.0 drop support for Kitty version < 0.32.2 + use Kitty's builtin bracketed paste + experimental tmux support"
featuredImage = 'kitty-scrollback-github.png'
images = ['kitty-scrollback-github.png']
+++

[kitty-scrollback.nvim v5.0.0](https://github.com/mikesmithgh/kitty-scrollback.nvim/releases/tag/v5.0.0) is officially released!

## What is kitty-scrollback.nvim?

A Neovim plugin (and Kitty Kitten) that allows you to navigate your Kitty scrollback buffer to quickly search, copy, and execute commands in Neovim.

![demo](https://preview.redd.it/v5-0-0-drop-support-for-kitty-version-0-32-2-use-kittys-v0-0iutp6z7a55d1.gif?width=1342&auto=webp&s=d3af4bf4b6d9c528d22a3a338d6830ec8713a0d8)

Check out the [README](https://github.com/mikesmithgh/kitty-scrollback.nvim) for detailed information, the [Wiki](https://github.com/mikesmithgh/kitty-scrollback.nvim/wiki) for additional configurations, and [Advanced Configuration Examples](https://github.com/mikesmithgh/kitty-scrollback.nvim/wiki/Advanced-Configuration-Examples) for more demos!

## What changed?

See [Migrating to v5.0.0](https://github.com/mikesmithgh/kitty-scrollback.nvim#-migrating-to-v500) for the detailed migration guide.

  - kitty-scrollback.nvim v5.0.0 uses Kitty's builtin `--bracketed-paste` option when sending commands to Kitty. The `--bracketed-paste` option was added in Kitty 0.32.2. If you are using an older version of Kitty, then upgrade to the latest version or at least 0.32.2.
  - Alternatively, if you are unable to upgrade Kitty, then you can still use tag [v4.3.6](https://github.com/mikesmithgh/kitty-scrollback.nvim/releases/tag/v4.3.6) of kitty-scrollback.nvim.
  - See [kitten-send-text](https://sw.kovidgoyal.net/kitty/remote-control/#kitten-send-text) for more information on the `--bracketed-paste` option.

## Other minor version updates

  - Added experimental tmux support in v4.0.3! See [tmux (🧪 experimental )](https://github.com/mikesmithgh/kitty-scrollback.nvim?tab=readme-ov-file#tmux--experimental-) for setup instructions.
  - A handful of bug fixes and smaller features, see the [CHANGELOG](https://github.com/mikesmithgh/kitty-scrollback.nvim/blob/main/CHANGELOG.md) for more information.

## What's next?
The next feature I plan to work on is adding command-line editing support (see [#253](https://github.com/mikesmithgh/kitty-scrollback.nvim/issues/253)). I also plan to 
complete the work necessary to move tmux support from experimental to stable.

If you have any questions, comments, or feedback feel free to create an [issue](https://github.com/mikesmithgh/kitty-scrollback.nvim/issues) or open a [discussion](https://github.com/mikesmithgh/kitty-scrollback.nvim/discussions).
