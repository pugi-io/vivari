# Vivari

When working across sibling Pugi repositories, follow the workspace-level policy if the enclosing multi-repository checkout is available. This file owns this repository's additional rules.

Vivari runs Node-style projects in the browser using workers, a virtual filesystem, a Node-compatible runtime, and WebAssembly modules. For runtime, worker, protocol, or networking changes, read [`ARCHITECTURE.md`](ARCHITECTURE.md) first. [`roadmap.md`](roadmap.md) records historical decisions; verify current behavior in source and tests.

## Architecture boundaries

- `packages/protocol/syscall.js` owns the worker/host shared-buffer ABI. Keep opcodes, layouts, and serialization in one source of truth; update both sides and protocol tests together.
- `packages/kernel-host/` owns process supervision, filesystem services, and network routing. `packages/runtime/` owns guest-process behavior and Node/Bun-compatible APIs. `packages/studio/` owns the browser IDE and preview. Rust crates under `packages/vfs`, `packages/codec`, and `packages/crypto` produce Wasm used by browser and Node test paths.
- Fix compatibility gaps at the layer that violates the real runtime contract. Reproduce the behavior against the actual guest/package and trace the boundary before adding special cases.
- Treat guest code and messages as untrusted. Validate at kernel/protocol boundaries, preserve process and tenant isolation, and never silently swallow syscall or transport failures.
- Host and guest globals/environments differ. Confirm which realm owns a value before reading env, globals, files, network, or timers inside a worker.

## Verification

- Check `node -v` against the manifest (`>=22.15.0`). Missing generated Wasm can make runtime tests fail misleadingly. Before Node-tier runtime verification, build the needed crates with `npm run build:vfs:node`, `npm run build:codec:node`, and/or `npm run build:crypto:node`; after Rust changes, also build the matching browser artifacts.
- `npm run verify` exercises the headless Node runtime. After protocol changes, run the relevant runtime and protocol tests. Use focused probes when changing the loader, builtin registry, egress, cookies, or terminal feedback; their names are listed in the root `package.json`.
- Build the Wasm/WASI prerequisites before relying on `npm run build` or spike results. Do not run vendor or network-producing scripts unless required by the task.
- Test the real entry path for user-visible behavior; a unit spike that bypasses the worker/UI boundary does not verify the whole flow. Report exact checks and limits.

## Changes

- Make the smallest change at the correct runtime, protocol, or UI owner. Preserve observable Node/Bun compatibility and cross-environment behavior.
- Update the existing architecture/module docs when a maintained contract changes. Consult only the relevant sections of [`docs/AGENT-ENGINEERING-KNOWLEDGE.md`](docs/AGENT-ENGINEERING-KNOWLEDGE.md) for prior implementation findings; those notes may be historical and must be checked against current source.
- Inspect Git status first and preserve local work. Avoid destructive Git operations. Do not commit, push, publish, or deploy without explicit authorization in the current request.
