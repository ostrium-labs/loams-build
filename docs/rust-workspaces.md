# Cargo workspaces under Buck2: what we learned

Notes from moving Loams' Rust workspaces (about 95 crates, 1 400 locked packages) to Buck2 with
reindeer, Cargo staying the source of truth. Each item cost real time; the Loams-side files are
named so the pattern can be reused.

## Third-party crates: a generated pseudo-package, not the workspace

Pointing reindeer's `manifest_path` at the workspace `Cargo.toml` has two problems: it buckifies
only the members' normal and build dependencies (no dev-dependencies, so tests cannot build), and
when two members use different major versions of a crate under the same name it writes two public
aliases with one name. Loams' `scripts/buck/gen.py` instead writes `third-party/rust/Cargo.toml`: one
dependency per package that any member uses directly (normal, build or dev), pinned with `=` to the
locked version, with exactly the features `cargo metadata` resolves, and keyed `name` or
`name-<semver major>` when there are several. It copies the workspace's `Cargo.lock` next to it, so
every transitive version stays the same, and checks that afterwards.

## First-party crates: one macro call per Cargo package

`buck/rust.bzl` (`cargo_package`) turns one package into its library, binaries, unit tests,
integration tests, examples, benches, build script and a clippy filegroup. The generator fills in
dependencies (with `select` for platform-specific ones, `named_deps` for renames), features, lints
from `[lints]`, and a crate's own `clippy.toml`.

## Paths

- **`buck-out` cannot be a symlink.** Buck2 links sources and outputs into `buck-out` with
  relative symlinks (`../../../crates/...`). If `buck-out` points to another disk, those resolve
  physically and break. Put the whole checkout on the big disk instead.
- **Lay sources out at their repository paths.** A `rust_filegroup` whose `mapped_srcs` puts
  `crates/foo/src/lib.rs` and, say, `release/x.toml` at those paths means
  `include_str!("../../../release/x.toml")` still resolves. The generator scans for `include*!`
  of files outside each target's directory; files in another package are exported from it.
- **`CARGO_MANIFEST_DIR` relative to the repository root.** The prelude makes it an absolute path
  into the symlinked sources in `buck-out`, so `env!("CARGO_MANIFEST_DIR")` joined with `../..`
  lands in `buck-out`, and an absolute path would be baked into binaries shared through a remote
  cache. A rustc wrapper sets it to `crates/foo` instead; tests run from the repository root, as
  Buck2 runs them, so the path reaches the checkout. `CARGO_BIN_EXE_*` and `CARGO_TARGET_TMPDIR`
  are made relative the same way.

## Toolchain

- **Pin rustup's proxy with `RUSTUP_TOOLCHAIN`.** Build scripts run inside `buck-out`; without the
  pin, a `rustc` they call may not see `rust-toolchain.toml`.
- **A tool must be a single argument.** The prelude's clippy wrapper formats `clippy_driver` with
  `"{} $@"`, one line per argument, so `RunInfo(args = [wrapper, "clippy-driver"])` breaks it.
  Use one small script per tool (`toolchains/rust/{rustc,clippy-driver,rustdoc}`).
- **`-Copt-level=0` in the toolchain flags** also gives build scripts `OPT_LEVEL`, which some
  (libm) unwrap.

## Build scripts

Most crates' `build.rs` only probes the compiler; the default fixup runs it and keeps any libraries
it links (`fixups/` here). Crates that build C with the `cc` crate work the same way, through the
prelude's `CC`/`CXX`/`AR` shims.

## Clippy and tests

- `[clippy.txt]` subtargets never fail a build; a wrapper (`scripts/buck/clippy.sh`) fails when any
  is non-empty, as `-D warnings` does.
- `buck2 test` counts test binaries. Summing libtest's `test result:` lines gives Cargo's counts.

## Build scripts that read outside their crate

The prelude's `buildscript_run` starts `build.rs` in a fresh directory holding only the crate's
top-level entries, so `../../proto` finds nothing. Loams wraps the build script
(`buck/run_build_script.sh`) to `cd` into a tree laid out at repository paths and set
`CARGO_MANIFEST_DIR` there, as Cargo would.

## Remote execution (NativeLink)

- **Ship the toolchain as an input.** A worker should not need rustup: rustc, rust-std and clippy
  for the `rust-toolchain.toml` channel are `http_archive`s pinned by the channel manifest's sha256,
  merged into one sysroot by a genrule. Remote inputs are read-only, so the merge must `chmod u+w`
  between copies.
- **Give actions a PATH.** Workers may start actions with no `PATH`; the linker wrapper
  (`#!/usr/bin/env bash`) then fails with status 127.
- **Pin workers by property** (`host-distro`) while the fleet is mixed.
- With deferred materialization, intermediate outputs stay in the CAS: local `buck-out` stays at a
  few GB even for a full workspace build.

## The file watcher

The default `notify` watcher puts an inotify watch on every directory under the project root,
`buck-out` included. With a full `buck-out` the daemon took six minutes to start under load and can
exhaust `fs.inotify.max_user_watches`. `file_watcher = fs_hash_crawler` (hash the source tree,
skipping `buck-out`, on each command) cost about a second on a 19 000-file tree; `watchman` is the
better choice where it is installed.
