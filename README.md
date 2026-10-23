# Raye Linux packages

Recipes for the [rpk](https://github.com/Raye-Linux/rpk) package repo.
Everything is built static against musl.

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

`zlib` and `openssl` are build-only: curl and git link them in, they
are never installed on their own.

The built repo is published as the `repo` release of this repository,
which is where `raye-update` fetches from.

## Building

`mkrepo` turns the recipes into a repo rpk can use. It needs a musl
compiler, which the live build already makes:

```sh
CC=/path/to/musl-gcc ./mkrepo repo/            # everything
CC=/path/to/musl-gcc ./mkrepo repo/ dropbear   # just one
```

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

Services go in `/etc/rc.d/<name>` and take `start` and `stop`.

## License

MIT, see [LICENSE](LICENSE). The packaged software keeps its own.
