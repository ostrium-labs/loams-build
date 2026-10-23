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
| Bun | runtime, test runner, package resolution, and **AOT packaging** (`bun build --compile --bytecode`) | MIT, but it statically links JavaScriptCore, which is LGPL-2. A compiled binary carries that runtime, so a distributed one needs the LGPL-2 notice and relinking path (Loams D1794, proposed; counsel check Q1312) |
| ts-rs | Rust-to-TypeScript types | MIT |
| OXC | parser, transformer, oxlint | MIT |
| Rolldown | bundler | MIT |
| Vite | dev server and bundling | MIT |
| Vitest | test runner | MIT |
| Vite+ (VoidZero) | **recommended toolchain**: dev, check (Oxlint, Oxfmt, types), test (Vitest), build (Vite, Rolldown), pack (tsdown), `vp run` | MIT (`voidzero-dev/vite-plus` LICENSE, read 2026-10-23); 1.0.0 stable since 2026-09-28 |
| SWC | where it fits | Apache-2.0 |
| Biome | where it fits | MIT OR Apache-2.0 |

**Vite+ is the recommended toolchain** (owner, 2026-10-23; Loams decision D1770). Plain Vite with Rolldown and Vitest stays the fallback, and our TypeScript rules must keep working with it: they call the standard tools that Vite+ bundles, not `vp`-only behaviour. Cloudflare acquired VoidZero in June 2026 and said Vite, Vitest, Rolldown, Oxc and Vite+ stay MIT; we re-read the LICENSE files before each upgrade. The Loams-side mapping is in [design 79: Void and Vite+](https://github.com/ostrium-labs/loams/blob/dev/docs/design/79-void-and-vite-plus.md) (approved; [ostrium-labs/loams#1108](https://github.com/ostrium-labs/loams/pull/1108)).

**Bun AOT is the way to package TypeScript outputs** (owner, 2026-10-23; Loams D1791). An app or tool becomes one executable with `bun build --compile --bytecode --minify --sourcemap --format=esm`, cross-compiled with `--target=bun-<os>-<arch>[-musl]`. Bytecode is tied to the Bun version that built it, so rules pin one Bun version per build. Native addons that can't be embedded for the target, and code that reads files beside itself, fall back to `bun run` or Node. Deno is not a target (Loams D1790).

We do not copy code under BSL-like or source-available licences, and running such a tool needs a check first.

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