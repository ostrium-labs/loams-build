# reindeer fixups for widely used crates

One directory per crate, as reindeer reads them (`fixups/<crate>/fixups.toml`). These are the
generic ones: they describe the crate, not Loams. A Loams workspace copies this directory to its
`third-party/rust/fixups` (and adds any Loams-specific fixups there).

Most crates' `build.rs` only probes the compiler or generates code, and the default is to run it
as Cargo does and keep whatever it links:

```toml
[buildscript.run]
rustc_link_lib = true
rustc_link_search = true
```

Crates that build C (ring, zstd-sys, lz4-sys, libsqlite3-sys, aws-lc-sys, blake3, rquickjs-sys,
libmimalloc-sys) work with the same default, through the prelude's `CC`/`CXX`/`AR` shims, as long
as the C++ toolchain names its compiler by absolute path (the shims exec it without a `PATH`
lookup). Exceptions carry a comment saying why, for example `aws-lc-rs`, whose build script
requires `links` metadata that Buck2 does not pass between build scripts.
