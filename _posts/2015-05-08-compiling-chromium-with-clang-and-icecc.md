---
layout: post
title: "Compiling Chromium with Clang and Icecream"
tags: [Chromium, clang, icecc]
---

[Icecream](https://github.com/icecc/icecream) lets you use the
computing power of nearby computers to speed up your compilation.  At
the end of this tutorial, your build time should go down substantially
(assuming the machines that will be helping you compile are
fast and close to you on the network). This guide is specific to
Chromium, but you can use Icecream to compile other projects too.

This guide expects that you can already compile Chromium locally
without Icecream, that somebody else already set up the `icecc`
scheduler, your colleagues are using it and you want to join them.
I'm personally using Linux Mint 17.1 Rebecca (based on Ubuntu Trusty),
but this shouldn't matter to you.

## Setting up Icecream

    $ sudo apt-get install icecc

Go to `/etc/icecc.conf` and edit `ICECC_NETNAME` to the network name
that was set up on the scheduler.

    $ sudo service iceccd restart

Now run the monitoring GUI:

    $ icemon -n ICECC_NETNAME  # change the netname to yours

You should be able to see other servers on the network, including your
own host name. If you don't, check these things:

1. Are you are using the correct network name (same as set on the
server with the scheduler)?
2. Is `iceccd` running? Try `service iceccd status`.
3. Is your computer is on the same network as the scheduler? Can you ping
it? Can you connect to it using the Icecream port (8765 by default)?
Try running `telnet <scheduler_ip> 6765`.

Once you have that, configure it to be used whenever the compiler gets called.
Add `icecc` binaries to the *beginning* of your path (also add it to your
`.bashrc` file):

    $ export PATH="/usr/lib/icecc/bin:$PATH"
    $ which gcc
    /usr/lib/icecc/bin/gcc
    $ which g++
    /usr/lib/icecc/bin/g++

If your results are different, check if the directory `/usr/lib/icecc/bin`
exists. You might have noticed that a `clang` binary isn't present in this
directory - don't worry, I'll get to that later.

## Chromium and Icecream without Clang

Clang takes some extra steps to set up, so let's get it running with
`gcc` first.

    $ export GYP_DEFINES="$GYP_DEFINES clang=0 linux_use_debug_fission=0 \
        linux_use_bundled_binutils=0"

Now try to compile Chromium (I'm going to assume it's located in
`~/chromium/` for the rest of the tutorial):

    $ cd ~/chromium/src  # or wherever your source is
    $ ninja -C out/Release -t clean
    $ gclient runhooks
    $ ninja -j 100 -C out/Release chrome

Now if you open `icemon -n ICECC_NETNAME`, you should see jobs leaving
from your node and being handled on other nodes. If you open the star
view (View->Mode->Star view), there should be an occasional dotted
line from your node (meaning that jobs are being sent out) to the
scheduler in the center, and a full line going to some of the other
nodes (jobs being received).

The number of jobs (I use 100) depends on how many CPUs are available
on your network. Try experimenting with them to see which number is
fastest for you.

## Adding Clang to the mix

There is extra setup involved here, since Chromium uses its own
patched version of Clang and Icecream didn't support Clang until
recently.

First, create a Clang toolchain from the patched Chromium version:

    $ /usr/lib/icecc/icecc-create-env --clang \
      ~/chromium/src/third_party/llvm-build/Release+Asserts/bin/clang \
      /usr/lib/icecc/compilerwrapper

On the list line, it should write something like `creating
dfc1acedfe857c45ce7b.tar.gz`. Rename it to `clang.tar.gz`
and put it into your chromium source tree (`~/chromium/src/`). Set the
`ICECC_VERSION` variable and also put it into your `~/.bashrc`.

    $ export ICECC_VERSION=$HOME/chromium/src/clang.tar.gz  # change to where your file is

The `icecc` directory doesn't contain clang by default, you have to add it
so that Clang gets redirected trough Icecream.

    $ cd /usr/lib/icecc/bin
    $ sudo ln -s /usr/bin/icecc ./clang
    $ sudo ln -s /usr/bin/icecc ./clang++

Check with `ls -l` if all the files in `/usr/lib/icecc/bin` link to
the same `icecc` executable.

Add the patched Chromium version of the Clang executable to your PATH
right after Icecream (change the Chromium source location if it's
different for you):

     $ export PATH="/usr/lib/icecc/bin:$HOME/chromium/src/third_party/llvm-build/Release+Asserts/bin/:$PATH


Set the Chromium `GYP_DEFINES` again (remove the previous definitions
I recommended):

    $ export GYP_DEFINES="$GYP_DEFINES clang=1 \
      make_clang_dir=/usr/lib/icecc clang_use_chrome_plugins=0 \
      linux_use_debug_fission=0 linux_use_bundled_binutils=0"

I also recommend setting the environment variable `ICECC_CLANG_REMOTE_CPP` to
true, which will make Icecream run the C preprocessor on the remote nodes and
only pre-process the `#include` directives locally. Without this, you'd get a
lot of false positive warnings from Clang, because it would only see the files
with expanded macros and report that, for example, you are using unnecessary
double parenthesis.

    $ export ICECC_CLANG_REMOTE_CPP=1

Clean your Chromium build and rebuild.

    $ ninja -C out/Release -t clean
    $ gclient runhooks
    $ ninja -j 100 -C out/Release chrome

Check again with the monitoring GUI if you can see the compiling tasks being
sent to other nodes. For more options, look at `icecc --help`.
