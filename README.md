# Stack Smash Lab 💥

An interactive, browser-based visualization of a **stack-frame buffer overflow** — built as a safe
educational module to help learners understand one of the most famous bug classes in computing history.

No real exploit, no shellcode, no attack tooling: all "memory" in this app is fake and lives only in the page.

## What it teaches

When a C function like this one is called:

```c
void greet(const char *input) {
    char buffer[16];
    strcpy(buffer, input);  // ← the bug: no bounds checking
    printf("Hello, %s!\n", buffer);
}
```

…the CPU builds a **stack frame**: space for `buffer` at low addresses, the caller's saved
**frame pointer (EBP)** above it, and the **return address** above that. Because `strcpy()` never
checks the destination size, an input longer than 15 characters keeps writing "upward" — first
corrupting the saved EBP, then overwriting the return address. When the function returns, the CPU
trusts whatever bytes are in that slot and jumps there.

This app lets you watch that happen byte by byte.

## How to run

It's a single static file — no build step, no dependencies.

- **Easiest:** open `index.html` in any modern browser (double-click it), **or**
- Serve the folder, e.g. `python -m http.server 8000` or `npx serve`, then visit `http://localhost:8000`.

Because it is plain HTML/CSS/JS, it also works as-is on **GitHub Pages**
(Settings → Pages → deploy from branch → `main` / root).

## Features & interactions

| Control | What it does |
| --- | --- |
| **Input box** | Type any string (up to 48 chars); a live preview shows how many bytes `strcpy` will write and warns when it exceeds 16. |
| **Scenario presets** | One-click demos: safe input, exact fit (15 chars + NUL), classic **off-by-one**, and a full stack smash. |
| **▶ Run strcpy()** | Animated byte-by-byte copy into the buffer, with a highlighted write cursor and narrated execution log. |
| **⏭ Step one byte** | Manual single-stepping for close study. |
| **Speed slider** | Controls the animation delay. |
| **🛡️ Stack canary toggle** | Inserts a canary between the buffer and saved EBP. Overflow the buffer and watch the program **abort safely** (`*** stack smashing detected ***`) instead of returning into corrupted memory. |
| **↩ Execute return** | Finishes the function: either returns cleanly to `main`, or jumps to the corrupted address and crashes with a simulated `SIGSEGV`. |
| **Live monitors** | Real-time hex view of the canary, saved EBP, and return address as your bytes overwrite them. |

The memory grid shows every byte's address, hex value, and ASCII character, color-coded:
green = in-bounds write, red = overflow write, amber = canary, purple = saved EBP, blue = return address.

## Learning highlights

- **Exact fit is safe:** 15 chars + the NUL terminator exactly fill `buffer[16]` — nothing overflows.
- **Off-by-one matters:** 16 chars + NUL writes one byte past the buffer, corrupting the first byte
  of the saved EBP (or tripping the canary). Tiny bug, real consequences.
- **The return address is the prize:** overwriting it hijacks control flow the moment the function returns.
- **Mitigations work:** the canary toggle demonstrates how compilers turned this catastrophic bug class
  into a (usually) safe crash, alongside ASLR, NX/DEP, and bounds-checked APIs.

## Repository structure

```
buffer-overflow-viz/
├── index.html      # the entire app (HTML + CSS + JS, no dependencies)
├── README.md       # this file
└── reflection.md   # reflection on the build process and what was learned
```

## Disclaimer

This is a **teaching simulation**. It does not compile, exploit, or attack anything — the "stack"
is a JavaScript model rendered in the page. It exists to build intuition about memory safety,
inspired by classic educational modules like [BOMOD](https://github.com/benjholla/bomod).

## License

MIT — free to use, remix, and extend for teaching.
