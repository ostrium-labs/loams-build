# loams-build

**Main goal: first-class TypeScript support in Buck2, built on the Rust-based JavaScript and TypeScript tooling ecosystem.**

This is Ostrium Labs' research fork of Buck2. It is meant to become the build engine for [qrc](https://github.com/ostrium-labs/qrc) Layers 3 and 4.

**Status: planned, not built.** Nothing here is released or supported yet, and every item below is a plan.

## Goals (all planned, not built)

- Buck2 rules that build, test and bundle TypeScript packages hermetically.
- `.d.ts` pipelining, so a dependent package can start as soon as its dependencies' declarations exist.
- Remote cache hits for TypeScript actions.
- Nx-style `affected` commands.

## Tools we plan to build on

Licences below were read from each project's LICENSE files on 2026-10-23.

| Tool | Role | Licence |
|---|---|---|
| Bun | runtime, test runner, package resolution | MIT, but it statically links JavaScriptCore, which is LGPL-2 (needs a check before we ship or bundle it) |
| ts-rs | Rust-to-TypeScript types | MIT |
| OXC | parser, transformer, oxlint | MIT |
| Rolldown | bundler | MIT |
| Vite | dev server and bundling | MIT |
| Vitest | test runner | MIT |
| Vite+ (VoidZero) | optional, on top of Vite | MIT (its repository and README say so); re-check before use |
| SWC | where it fits | Apache-2.0 |
| Biome | where it fits | MIT OR Apache-2.0 |

The open default is plain Vite with Rolldown and Vitest. Vite+ is optional. VoidZero has talked about commercial offerings, so we re-check the terms before depending on anything from them. We do not copy code under BSL-like or source-available licences, and running such a tool needs a check first.

## How we work: upstream-first

- Upstream's `main` branch is left untouched and tracks `facebook/buck2`.
- A change that is not specific to us goes to facebook/buck2 first.
- This fork keeps only a small patch queue on `loams/main`. We rebase it onto upstream regularly.

## Licence

This fork's licence is MIT OR Apache-2.0, as upstream. We keep upstream's `LICENSE-MIT`, `LICENSE-APACHE` and `NOTICE` files, and we mark the files we change.

## Name

"Buck2" is Meta's name. We do not use it for our own builds, and we do not imply that Meta endorses this fork.

## Links

- qrc: https://github.com/ostrium-labs/qrc
- Layer 3 plan: https://github.com/ostrium-labs/qrc/blob/main/docs/layer-3-scheduler.md