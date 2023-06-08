### THIS IS MY PERSONAL FORK OF CLIPMENU
https://github.com/cdown/clipmenu

clipmenu is a simple clipboard manager using [fzf][] and [xsel][].

## Installation

`clipmenud` uses [clipnotify](https://github.com/cdown/clipnotify), which is provided 
with this repo as a submodule. You will also need [fzf][] and [xsel][] installed.

clone the repo with submodules:

    git clone --recurse-submodules git@github.com:xero/clipmenu.git
    cd clipmenu

build clipnotify, then the clipmenu bins:

    cd clipnotify
    make && make install
    cd ..
    make install

finally `re{load,start}` the systemd unit:

    systemctl --user daemon-reload
    systemctl --user restart clipmenud.service

# Usage

## clipmenud

Start `clipmenud`, then run `clipmenu` to select something to put on the
clipboard. For systemd users, a user service called `clipmenud` is packaged as
part of the project.

For those using a systemd unit and not using a desktop environment which does
it automatically, you must import `$DISPLAY` so that `clipmenud` knows which X
server to use. For example, in your `~/.xinitrc` do this prior to launching
clipmenud:

    systemctl --user import-environment DISPLAY

if you're in a headless environment you need to create a "fake" xorg server for
[xsel][] to communicate with. i suggest using [xvfb][]. put something like this
in your shell init (e.g. `~/.zlogin` or `~/.bash_login`):

    if ! pgrep -x "Xvfb" >/dev/null; then
        export DISPLAY=:0
        Xvfb :0 -screen 0 1x1x8 &
        systemctl --user import-environment DISPLAY
        systemctl --user restart clipmenud.service
    fi

## clipmenu

this fork of clipmenu uses [fzf][] exclusivly. you can use it's environment vars
to setup the fzf style to your liking. i personally have the following set in my env:

    export FZF_DEFAULT_OPTS=$FZF_DEFAULT_OPTS' --color=fg:#c1c1c1,bg:#2b2b2b,hl:#5f8787 --color=fg+:#ffffff,bg+:#1c1c1c,hl+:#3ea3a3 --color=info:#87875f,prompt:#87875f,pointer:#8787af --color=marker:#8787af,spinner:#8787af,header:#5f8787 --color=gutter:#2b2b2b,border:#222222 --padding=1 --prompt=❯ --marker=❯ --pointer=❯ --reverse'

For a full list of environment variables that clipmenud can take, please see
`clipmenud --help`.

# Features

The behavior of `clipmenud` can be customized through environment variables.
Despite being only <300 lines, clipmenu has many useful features, including:

* Customising the maximum number of clips stored (default 1000)
* Disabling clip collection temporarily with `clipctl disable`, reenabling with
  `clipctl enable`
* Not storing clipboard changes from certain applications, like password
  managers
* Taking direct ownership of the clipboard
* ...and much more.

Check `clipmenud --help` to view all possible environment variables and what
they do. If you manage `clipmenud` with `systemd`, you can override the
defaults by using `systemctl --user edit clipmenud` to generate an override
file.

# Supported launchers

this fork is designed to only work with `CM_LAUNCHER` set to `fzf` (default)

# How does it work?

clipmenud is less than 300 lines, and clipmenu is less than 100, so hopefully
it should be fairly self-explanatory. However, at the most basic level:

## clipmenud

1. `clipmenud` uses [clipnotify](https://github.com/cdown/clipnotify) to wait
   for new clipboard events.
2. If `clipmenud` detects changes to the clipboard contents, it writes them out
   to the cache directory and an index using a hash as the filename.

## clipmenu

1. `clipmenu` reads the index to find all available clips.
2. `fzf` is executed to allow the user to select a clip.
3. After selection, the clip is put onto the PRIMARY and CLIPBOARD X
   selections. or stdout using the `CM_OUTPUT_CLIP=1` environment var.

# example

checkout my [dotfiles][] repo to see it in action.

[fzf]: https://github.com/junegunn/fzf 
[xsel]: http://www.vergenet.net/~conrad/software/xsel/
[xvfb]: https://www.x.org/releases/X11R7.6/doc/man/man1/Xvfb.1.xhtml
[dotfiles]: https://git.io/.files
