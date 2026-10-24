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
| desktop | all of the below that a session needs, plus Raye's defaults in /etc/xdg; logging in on tty1 starts Hyprland |
| hyprland | the Hyprland compositor, `hyprctl` with it |
| foot | a fast, light wayland terminal |
| mesa | OpenGL, GLES and EGL: Intel's iris, softpipe for anything else |
| eudev | udevd, so the compositor knows what the input devices are |
| seatd | lets the video group at the GPU and input without root |
| font-dejavu | one font family that covers most scripts, mono included |
| pipewire | sound: PipeWire, its Pulse server and ALSA plugin |
| wireplumber | PipeWire's session manager |
| dbus | the message bus, and a service for the system bus |
| flatpak | sandboxed apps from Flathub and the like (`flatpak --user`) |
| xdg-desktop-portal | file choosers, screenshots and screen sharing for sandboxed apps, with the -hyprland and -gtk backends |
| gtk3 | GTK 3, wayland only |
| waybar | the status bar |
| rofi | launcher and menus |
| swaync | notifications, and a centre to look back at them (the desktop's) |
| mako | a plainer notification daemon, if you'd rather |
| gtk4, libadwaita | GTK 4 and GNOME's widgets on it, wayland only, for swaync and Flatpak-less GTK 4 apps |
| hyprlock, hypridle, hyprpaper | lock screen, idle daemon, wallpapers |
| grim, slurp, wl-clipboard | screenshots, picking a region, the clipboard |

Everything under Hyprland is dynamically linked: glib, cairo, pango,
wayland, libinput, the hyprwm libraries and so on, each its own package.
`rpk install desktop` brings them in.

`openssl`, `ncurses`, `readline`, `libnl` and `libevent` are build-only, along
with headers-only things like `wayland-protocols`: whatever needs them
links them in or compiles them in, they are never installed on their own.

The built repo is published as snapshots, pre-releases of this
repository named after the day they were built, and that's where
`raye-update` fetches from (see Publishing).

## Categories

Recipes are sorted the Gentoo way, `pkg/<category>/<name>/`, with
Gentoo's category names wherever one fits: `sys-libs/musl`,
`dev-libs/glib`, `media-libs/mesa`, `gui-wm/hyprland`,
`net-wireless/wpa_supplicant`. Dependencies are written with the full
name. On the command line, mkrepo and rpk take the name alone too, when
only one category has it. `raye/` is the one category of Raye's own,
for packages that are mostly Raye's configuration (`raye/desktop`, the
defaults the desktop profile comes with).

## Profiles

`profiles/` has what a system can be made into, one directory each:
`packages` (one per line, full names), `parent` (the profile it builds
on, if any) and `description`. mkrepo turns them into `PROFILES` next to
the INDEX, which is what `rpk profile` reads.

| profile | what for |
|---|---|
| minimal | raye-base (the boot scripts and /etc), busybox, rpk, the kernel, curl and sudo: base.tar on the live image |
| desktop/hyprland | minimal and the Hyprland desktop, with Raye's defaults |
| toolchain | minimal, gcc, binutils, make and pkgconf, to build recipes on raye |

## Publishing

`snapshot` takes a built repo and an ed25519 key, and makes a signed
copy of it in a new directory named after the snapshot: the INDEX and
PROFILES with their signatures, the tarballs and a `SHA256SUMS`. That
directory is a repo as it stands, for any web server.

Snapshots belong to a release, the first number of the Raye version
they're for (0 for every 0.0.x, 1 for every 1.x), and are named
`snapshot-<release>-<date>`. Next to the new directory,
`latest-<release>/SNAPSHOT` gets the snapshot's name, which makes the
folder around it a mirror: what `raye-update` asks first is
`<mirror>/latest-<release>/SNAPSHOT` for its system's release, then it
fetches `<mirror>/<snapshot>/INDEX` and the rest. A system stays on its
release until `RELEASE=` in its `/etc/rpk.conf` moves it.

With `-r snapshot-<release>-<date>` the snapshot also goes up as a
pre-release here, through the `gh` tool, and the pre-release called
`latest-<release>` gets a new `SNAPSHOT` naming it. GitHub is then a mirror with the same layout,
at `https://github.com/Raye-Linux/packages/releases/download`, and every
system finds the new snapshot on its next `raye-update`, with nothing
to edit.

```sh
openssl genpkey -algorithm ed25519 -out raye.key    # once
./snapshot -r snapshot-0-2026-10-24 ../live/build/repo raye.key mirror/snapshot-0-2026-10-24
```

For a mirror of your own, serve `mirror/` as it is, nginx with `root`
pointing at it is enough, and keep the older snapshots and other
releases' in it or not.
Systems ask the mirrors in `MIRRORS=` in their `/etc/rpk.conf` first,
then the ones in `/usr/share/raye/mirrors`, which raye-base installs
(from live's `rootfs/base`), so adding a mirror there reaches everyone
with their next update.

Systems trust the key once `raye.pub` (the public half, also left in
the directory as `key.pub`) is in their `/etc/rpk/keys`.

## The first snapshot of a release

Nothing is published for a release until its first snapshot is, and
until then `raye-update` says "no snapshot published for release 0
yet". For release 0 (every 0.0.x), once:

0. The project key, made once and kept for good. Whoever has the
   private half is who can publish packages every install accepts, and
   losing it means installs can't be told about a new one. Make it
   somewhere that stays (not a throwaway machine), and keep a copy
   offline:

   ```sh
   umask 077
   openssl genpkey -algorithm ed25519 -out raye.key
   openssl pkey -in raye.key -pubout -out raye.pub
   ```

   Commit `raye.pub` (only that) as `rootfs/base/etc/rpk/keys/raye.pub`
   in live. raye-base takes it to every install with its next update,
   and from then on mklive won't sign with any other key.

1. Build, signed with that key:

   ```sh
   cd live
   RPK_KEY=raye.key ./mklive packages
   ```

2. Publish it, as a pre-release and as a mirror folder:

   ```sh
   d=snapshot-0-$(date +%F)
   ../packages/snapshot -r $d build/repo raye.key out/mirror/$d
   ```

3. Check that GitHub has it, and copy `out/mirror` to the CDN if there's
   one:

   ```sh
   curl -fsSL https://github.com/Raye-Linux/packages/releases/download/latest-0/SNAPSHOT
   curl -fsSL -o /dev/null -w '%{http_code}\n' https://github.com/Raye-Linux/packages/releases/download/$d/INDEX
   ```

   The first prints `$d`, the second `200`.

4. On an install from before the mirrors (its `raye-update` reads
   `MIRROR=`, and has no `/usr/share/raye/mirrors`), update once with
   the new rpk straight from the snapshot, as root:

   ```sh
   curl -fLo /tmp/rpk https://raw.githubusercontent.com/Raye-Linux/rpk/main/rpk
   env RPK_REPO=https://github.com/Raye-Linux/packages/releases/download/$d sh /tmp/rpk update
   ```

   That brings raye-base (the new `raye-update`, the mirror list and
   `raye.pub`) and the new rpk with everything else. An install without
   a key in `/etc/rpk/keys` takes that first update on the sha256 sums
   alone, over https; from the next one on it checks the signature. It puts a new kernel in
   `/boot/raye.efi` but not on the EFI partition, which is
   raye-update's job, so copy it over the one you boot, keeping the
   old one:

   ```sh
   cp /boot/efi/EFI/BOOT/BOOTX64.EFI /boot/efi/EFI/BOOT/BOOTX64.EFI.old
   cp /boot/raye.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
   ```

   (or `EFI/raye/raye.efi`, wherever the install put it). From then on
   `raye-update` is all it takes.

## Building

`mkrepo` turns the recipes into a repo rpk can use. It needs a musl
toolchain, which the live build already makes:

```sh
CC=/path/to/x86_64-linux-musl-gcc ./mkrepo repo/            # everything
CC=/path/to/x86_64-linux-musl-gcc ./mkrepo repo/ dropbear   # just one
./mkrepo -p repo/                                           # INDEX and PROFILES only
```

Whatever is asked for gets built along with everything it depends on,
each package after its dependencies. A package already in the repo at
the recipe's version is left alone. `CXX` defaults to the `g++` next to
`CC`. Dynamically linked packages also need `sys-libs/musl` and
`sys-libs/gcc-libs` in the repo directory, the live build puts them
there.

mkrepo runs on raye too, with the toolchain profile, and that's how
`rpk` builds packages when `SOURCE=build`. There, recipes built with
meson or cmake stop with a message saying which tool isn't there. The desktop packages need more on the build machine
than the rest, the live repo's README has the list.

Normally you don't call it yourself, `./mklive packages` in
[Raye-Linux/live](https://github.com/Raye-Linux/live) does.

## Writing a recipe

A recipe is `pkg/<category>/<name>/build`, a shell file:

```sh
version=1.2.3
release=1
description="what it is, in a few words"
depends="sys-libs/musl and other packages, space separated"
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
it links in `depends` (`sys-libs/musl`, `sys-libs/gcc-libs` for C++):
mkrepo reads the binaries and won't pack one that needs something
nothing in there provides.

Services go in `/etc/rc.d/<name>` and take `start` and `stop`.

## License

MIT, see [LICENSE](LICENSE). The packaged software keeps its own.
