<!-- elide-retirement:2026-11-08 -->
> [!IMPORTANT]
> **Archived by Elide.** Elide's fork of this project was retired on November 8, 2026 and is no longer maintained here. Its code, branches, and history remain available for reference. Upstream project: https://github.com/swiftwasm/uwasi.
>
> Explore our current projects: [myna](https://github.com/elide-dev/myna), [bemo](https://github.com/elide-dev/bemo), [trufflecel](https://github.com/elide-dev/trufflecel), [truffledata](https://github.com/elide-dev/truffledata), and [elide](https://github.com/elide-dev/elide).

[![npm version](https://badge.fury.io/js/uwasi.svg)](https://badge.fury.io/js/uwasi)
[![.github/workflows/test.yml](https://github.com/swiftwasm/uwasi/actions/workflows/test.yml/badge.svg)](https://github.com/swiftwasm/uwasi/actions/workflows/test.yml)

# μWASI

This library provides a WASI implementation for Node.js and browsers in a tree-shaking friendly way.
The system calls provided by this library are configurable.

With minimal configuration, it provides WASI system calls which just return `WASI_ENOSYS`.

## Features

- No dependencies
- Tree-shaking friendly
  - 3 KB when minimal configuration
  - 6 KB when all features enabled
- Almost compatible interface with [Node.js WASI implementation](https://nodejs.org/api/wasi.html)
- Well tested, thanks to [wasi-test-suite by Casper Beyer](https://github.com/caspervonb/wasi-test-suite)

## Installation

```bash
npm install uwasi
```

## Example

### With all system calls enabled

```js
import { WASI, useAll } from "uwasi";
import fs from "node:fs/promises";

async function main() {
    const wasi = new WASI({
        args: process.argv.slice(2),
        features: [useAll()],
    });
    const bytes = await fs.readFile(process.argv[2]);
    const { instance } = await WebAssembly.instantiate(bytes, {
        wasi_snapshot_preview1: wasi.wasiImport,
    });
    const exitCode = wasi.start(instance);
    console.log("exit code:", exitCode);

/* With Reactor model
    wasi.initialize(instance);
*/
}

main()
```

### With no system calls enabled

```js
import { WASI, useAll } from "uwasi";

const wasi = new WASI({
    features: [],
});
```

### With `environ`, `args`, `clock`, `proc`, and `random` enabled

```js
import { WASI, useArgs, useClock } from "uwasi";

const wasi = new WASI({
    args: ["./a.out", "hello", "world"],
    features: [useEnviron, useArgs, useClock, useProc, useRandom()],
});
```

### With `fd` (file descriptor) enabled only for stdio

By default, `stdin` behaves like `/dev/null`, `stdout` and `stderr` print to the console.

```js
import { WASI, useStdio } from "uwasi";

const wasi = new WASI({
    features: [useStdio()],
});
```

You can use custom backends for stdio by passing handlers to `useStdio`.

```js
import { WASI, useStdio } from "uwasi";

const inputs = ["Y", "N", "Y", "Y"];
const wasi = new WASI({
    features: [useStdio({
        stdin: () => inputs.shift() || "",
        stdout: (str) => document.body.innerHTML += str,
        stderr: (str) => document.body.innerHTML += str,
    })],
});
```

## Implementation Status

Some of WASI system calls are not implemented yet. Contributions are welcome!

| Syscall | Status | Notes |
|-------|----------|---------|
| `args_XXX` | ✅ | |
| `clock_XXX` | ✅ | Monotonic clock is unavailable due to JS API limitation |
| `environ_XXX` | ✅ | |
| `fd_XXX` | 🚧 | stdin/stdout/stderr are supported |
| `path_XXX` | ❌ | |
| `poll_oneoff` | ❌ | |
| `proc_XXX` | ✅ | |
| `random_get` | ✅ | |
| `sched_yield` | ❌ | |
| `sock_XXX` | ❌ | |
