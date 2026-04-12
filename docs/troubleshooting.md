# Troubleshooting

Known gotchas when working with OPNET tooling, and their fixes.

---

## `@btc-vision/op-vm` — `GLIBC_2.39 not found` on Linux

### Symptom

Running contract unit tests (e.g. via `@btc-vision/unit-test-framework`) fails at startup with:

```
Failed to load native module: /lib/x86_64-linux-gnu/libc.so.6:
version `GLIBC_2.39' not found (required by
node_modules/@btc-vision/op-vm/prebuilds/linux-x64/op-vm.node)
Error: ... code: 'ERR_DLOPEN_FAILED'
```

### Cause

`@btc-vision/op-vm` ships prebuilt native binaries in `prebuilds/linux-x64/op-vm.node`. The current prebuild is linked against **glibc 2.39**, which ships with Ubuntu 24.04 (Noble) and newer. Any older distro hits `ERR_DLOPEN_FAILED`:

| Distro | glibc | Works with prebuild? |
|--------|-------|----------------------|
| Ubuntu 24.04 Noble | 2.39 | Yes |
| Ubuntu 22.04 Jammy | 2.35 | **No** |
| Debian 12 Bookworm | 2.36 | **No** |
| Debian 11 Bullseye | 2.31 | **No** |

Check yours with `ldd --version`.

### Fix: build op-vm from source against your system glibc

op-vm is a Rust crate exposed to Node via [Neon](https://neon-rs.dev/). Its `install.mjs` supports a build-from-source path that links the `.node` binary against whatever glibc is on your machine.

**Requirements:**

- Node.js **≥ 22**
- Rust toolchain — install via [rustup.rs](https://rustup.rs/)
- Build tools — `sudo apt-get install build-essential` on Debian/Ubuntu

**Option A — force it for a single install (recommended):**

```bash
npm_config_build_from_source=true npm install
```

This sets a flag `install.mjs` checks, skipping the prebuild and running `cargo build --release` via [`@neon-rs/cli`](https://neon-rs.dev/). The resulting `index.node` is placed at the root of the op-vm package and takes precedence over `prebuilds/`.

**Option B — rebuild only op-vm after a normal install:**

```bash
cd node_modules/@btc-vision/op-vm
rm -f index.node prebuilds/linux-x64/op-vm.node
npm run build
```

Under pnpm the path is `node_modules/.pnpm/@btc-vision+op-vm@<version>/node_modules/@btc-vision/op-vm`.

### Verification

After the build, the test runner should load op-vm without the glibc error. You can also confirm the binary is linked against your system glibc:

```bash
strings node_modules/@btc-vision/op-vm/index.node | grep -oE 'GLIBC_2\.[0-9]+' | sort -u
```

The highest version listed should be ≤ your system glibc (check with `ldd --version`).

### Alternative: run in a container

If you'd rather not install Rust on the host, run tests inside an Ubuntu 24.04 container where the prebuild works as-is:

```bash
docker run --rm -v "$PWD":/app -w /app node:22-noble npm test
```

### Upstream

Prebuilds live in [btc-vision/op-vm](https://github.com/btc-vision/op-vm) under `prebuilds/`. A prebuild linked against an older glibc (e.g. 2.31) would cover most LTS Linux releases — consider filing an issue upstream if this keeps biting contributors.
