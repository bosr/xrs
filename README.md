# Docker images for Rust cross-compilation

This project provides Docker images for cross-compiling your Rust projects.
Each image can be used on a Linux or macOS host and is dedicated to
cross-compiling to some target.

Rust 1.50 is the release currently used.

The images are derived from those of the awesome [Dockcross
project](https://github.com/dockcross/dockcross), simply adding `rustup`,
`cargo`, and some cross-compile target configured and ready to go.

All available targets are listed in the [table](#targets-table) below.


## Usage

Ensure your user has access to a running docker daemon, and determine your
cross-compiling target.


### Initial configuration

Assuming your target is `aarch64-unknown-linux-gnu`, execute the following in
order to create a bridging script.

    docker run --rm bosr/xrs:linux-arm64 > ./xrs-aarch64-unknown-linux-gnu
    chmod u+x ./xrs-aarch64-unknown-linux-gnu

This script will act as a transparent bridge to the cross-compiling toolchain.
You can move it to any location.

There is nothing special about the name of the script, I just use it to remind
me of the [target
triple](https://doc.rust-lang.org/nightly/rustc/platform-support.html) brought
by the `bosr/xrs:linux-arm64` image.

If you have other targets to cross-compile to, you can prepare other bridging
scripts, like for instance

    docker run --rm bosr/xrs:linux-arm64-musl > ./xrs-aarch64-unknown-linux-musl


### How to cross-compile a Cargo project

Let's remember that, assuming you have an existing Cargo project such as

    cargo init --bin hello && cd hello

building it with

    cargo build --release

on your host yields the following `target` tree structure

    target
    ├── CACHEDIR.TAG
    └── release
        ├── build
        ├── deps
        │   ├── hello-2adbcecc085a4b9a
        │   └── hello-2adbcecc085a4b9a.d
        ├── examples
        ├── hello      <---- binary for your host platform
        ├── hello.d
        └── incremental

Check the executable with

    $ file target/release/hello
    target/release/hello: Mach-O 64-bit executable x86_64

In order to cross-compile, you simply prepend a bridging script such as the one
created in the Initial configuration section,

    /path/to/xrs-aarch64-unknown-linux-gnu cargo build --release

Behind the scenes, the current directory is mounted as a volume in a temporary
Docker container, and your Cargo project is cross-compiled using the target
configured in the Docker image. This adds the cross-compiled target to the
`target` folder:

    target
    ├── CACHEDIR.TAG
    ├── aarch64-unknown-linux-gnu
    │   ├── CACHEDIR.TAG
    │   └── release
    │       ├── build
    │       ├── deps
    │       │   ├── hello-e2d5068318560224
    │       │   └── hello-e2d5068318560224.d
    │       ├── examples
    │       ├── hello
    │       ├── hello.d
    │       └── incremental
    └── release
        ├── build
        ├── deps
        │   ├── hello-2adbcecc085a4b9a
        │   └── hello-2adbcecc085a4b9a.d
        ├── examples
        ├── hello
        ├── hello.d
        └── incremental

You can check the resulting executable

    $ file target/aarch64-unknown-linux-gnu/release/hello
    target/aarch64-unknown-linux-gnu/release/hello: ELF 64-bit LSB pie executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, stripped

As a reminder, this target is `aarch64-unknown-linux-gnu` because because the
bridging script uses the Docker image `x-rs/linux-arm64`. Other available
targets are listed in a [table](#targets-table) down below.


If you also have created the bridging script for `aarch64-unknown-linux-musl`,
and you run

    /path/to/xrs-aarch64-unknown-linux-musl cargo build --release

then the final `target` tree structure is

    target
    ├── CACHEDIR.TAG
    ├── aarch64-unknown-linux-gnu
    │   ├── CACHEDIR.TAG
    │   └── release
    │       ├── build
    │       ├── deps
    │       │   ├── hello-e2d5068318560224
    │       │   └── hello-e2d5068318560224.d
    │       ├── examples
    │       ├── hello
    │       ├── hello.d
    │       └── incremental
    ├── aarch64-unknown-linux-musl
    │   ├── CACHEDIR.TAG
    │   └── release
    │       ├── build
    │       ├── deps
    │       │   ├── hello-0df44d73016fd116
    │       │   └── hello-0df44d73016fd116.d
    │       ├── examples
    │       ├── hello
    │       ├── hello.d
    │       └── incremental
    └── release
        ├── build
        ├── deps
        │   ├── hello-2adbcecc085a4b9a
        │   └── hello-2adbcecc085a4b9a.d
        ├── examples
        ├── hello
        ├── hello.d
        └── incremental



## <a name="targets-table"></a>Available cross-compilation targets


| Image name                 | target triple                  | dockcross base image        |
| ---                        | ---                            | ---                         |
| bosr/xrs:linux-arm64       | aarch64-unknown-linux-gnu      | dockcross/linux-arm64       |
| bosr/xrs:linux-arm64-musl  | aarch64-unknown-linux-musl     | dockcross/linux-arm64-musl  |
| bosr/xrs:linux-armv5       | armv5te-unknown-linux-gnueabi  | dockcross/linux-armv5       |
| bosr/xrs:linux-armv5-musl  | armv5te-unknown-linux-musleabi | dockcross/linux-armv5-musl  |
| bosr/xrs:linux-armv6       | arm-unknown-linux-gnueabihf    | dockcross/linux-armv6       |
| bosr/xrs:linux-armv6-musl  | arm-unknown-linux-musleabihf   | dockcross/linux-armv6-musl  |
| bosr/xrs:linux-armv7       | armv7-unknown-linux-gnueabihf  | dockcross/linux-armv7       |
| bosr/xrs:linux-armv7l-musl | armv7-unknown-linux-musleabihf | dockcross/linux-armv7l-musl |
| bosr/xrs:linux-mips        | mips-unknown-linux-gnu         | dockcross/linux-mips        |
| bosr/xrs:linux-mipsel      | mipsel-unknown-linux-gnu       | dockcross/linux-mipsel      |

All credits to [Dockcross project](https://github.com/dockcross/dockcross).


## Building the image

If you want to derive this project, rebuild the images, etc, here is how to
build the images by yourself.

    docker build -t bosr/xrs:linux-arm64      -f Dockerfile.linux-arm64      .
    docker build -t bosr/xrs:linux-arm64-musl -f Dockerfile.linux-arm64-musl .

    docker run --rm bosr/xrs:linux-arm64 > ./xrs-aarch64-unknown-linux-gnu
    ...
    ...

