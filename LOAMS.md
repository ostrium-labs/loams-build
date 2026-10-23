# loams-build

This is Ostrium Labs' research fork of Buck2. It is meant to become the build engine for [qrc](https://github.com/ostrium-labs/qrc) Layers 3 and 4.

**Status: planned, not built.** Nothing here is released or supported yet.

## How we work: upstream-first

- Upstream's `main` branch is left untouched and tracks `facebook/buck2`.
- A change that is not specific to us goes to facebook/buck2 first.
- This fork keeps only a small patch queue on `loams/main`. We rebase it onto upstream regularly.

## Licence

The licence is MIT OR Apache-2.0, as upstream. We keep upstream's `LICENSE-MIT`, `LICENSE-APACHE` and `NOTICE` files, and we mark the files we change.

## Name

"Buck2" is Meta's name. We do not use it for our own builds, and we do not imply that Meta endorses this fork.

## Links

- qrc: https://github.com/ostrium-labs/qrc
- Layer 3 plan: https://github.com/ostrium-labs/qrc/blob/main/docs/layer-3-scheduler.md