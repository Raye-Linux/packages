# Raye Linux packages

Recipes for the [rpk](https://github.com/Raye-Linux/rpk) package repo.
Built against musl, and static unless a recipe says otherwise.

| package | what for |
|---|---|
| dropbear | ssh server and client (`ssh` is a link to `dbclient`) |
| e2fsprogs | mkfs.ext4, e2fsck, tune2fs, resize2fs |
| util-linux | fdisk and sfdisk with GPT, lsblk, wipefs |
| fastfetch | system info, with the Raye eye as its logo |
| curl | downloads over https (OpenSSL), what raye-update uses |
| ca-certificates | the CA bundle TLS checks servers against |
| git | version control, https remotes through curl |
| gh | GitHub from the command line |
| bash | the GNU shell, at `/bin/bash` for scripts that want it |
| nano | text editor, syntax colours on |
| htop | interactive process viewer |
| tmux | terminal multiplexer: panes, windows, detachable sessions |
| ncdu | what's using your disk, interactively |
| rsync | fast copies, locally or over ssh |
| jq | JSON on the command line |
| tree | directory listings as a tree |
| terminfo | terminal descriptions (console, serial, xterm, kitty, tmux...) |
| wpa_supplicant | Wi-Fi: WPA2, WPA3 and enterprise, starts on every wireless interface |
| hyprland | the Hyprland compositor, `hyprctl` with it |
| foot | a fast, light wayland terminal |
| mesa | OpenGL, GLES and EGL: Intel's iris, softpipe for anything else |
| eudev | udevd, so the compositor knows what the input devices are |
| seatd | lets the video group at the GPU and input without root |
| font-dejavu | one font family that covers most scripts, mono included |

Everything under Hyprland is dynamically linked: glib, cairo, pango,
wayland, libinput, the hyprwm libraries and so on, each its own package.
`rpk install hyprland foot` brings them in.

`openssl`, `ncurses`, `readline`, `libnl` and `libevent` are build-only, along
with headers-only things like `wayland-protocols`: whatever needs them
links them in or compiles them in, they are never installed on their own.

The built repo is published as the `repo` release of this repository,
which is where `raye-update` fetches from.

## Building

`mkrepo` turns the recipes into a repo rpk can use. It needs a musl
toolchain, which the live build already makes:

```sh
CC=/path/to/x86_64-linux-musl-gcc ./mkrepo repo/            # everything
CC=/path/to/x86_64-linux-musl-gcc ./mkrepo repo/ dropbear   # just one
```

`CXX` defaults to the `g++` next to `CC`. Dynamically linked packages
also need `musl` and `gcc-libs` in the repo directory, the live build
puts them there. The desktop packages need more on the build machine
than the rest, the live repo's README has the list.

Normally you don't call it yourself, `./mklive packages` in
[Raye-Linux/live](https://github.com/Raye-Linux/live) does.

## Writing a recipe

A recipe is `pkg/<name>/build`, a shell file:

```sh
version=1.2.3
release=1
description="what it is, in a few words"
depends="other packages, space separated"
source="https://example.org/thing-$version.tar.gz"
sha256=...

build() {
    ./configure --prefix=/usr
    make -j"$JOBS"
    make DESTDIR="$pkg" install
}
```

`build()` runs inside the unpacked source. `makedepends` lists what it
needs to build (including `buildonly=1` libraries), and `$stage` holds
their headers and static libraries. `$pkg` is where to install,
`$files` is the recipe's own directory (for service scripts and such)
and `$JOBS` is the make job count. Afterwards mkrepo strips the
binaries, drops man pages, headers and static libraries, and packs what
is left. Bump `release` when the recipe changes but the version doesn't.

Static stays the default, and mkrepo refuses a static recipe that comes
out dynamically linked. `dynamic=1` is for things that really want
shared libraries (anything graphical, plugins, Mesa). To their build
systems those are cross builds, so pass `--host="$TARGET"` to configure,
`--cross-file "$MESON_CROSS"` to meson or
`-DCMAKE_TOOLCHAIN_FILE="$CMAKE_TOOLCHAIN"` to cmake. List every library
it links in `depends` (`musl`, `gcc-libs` for C++): mkrepo reads the
binaries and won't pack one that needs something nothing in there
provides.

Services go in `/etc/rc.d/<name>` and take `start` and `stop`.

## License

MIT, see [LICENSE](LICENSE). The packaged software keeps its own.
