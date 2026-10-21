# Dictate Programming Language

**Dictate** is an imperative, structured programming language built on deterministic line pacing, strict domain boundaries, and explicit mutation wrappers. Programs begin execution strictly through `mean start()`, with statement advancement authorized line-by-line via the `pass` directive.

---
## If you'd like to join a similar community!
[![Discord](https://img.shields.io/badge/Discord-Join%20Server-5865F2?style=flat&logo=discord&logoColor=white)]([https://discord.gg/YOUR_INVITE_CODE](https://discord.gg/4t9UctqEmX))

## Key Language Mechanics

* **Gated Pacing (`pass`):** A semicolon (`;`) terminates an instruction, but execution only advances if authorized by `pass`. Omitting `pass` causes the interpreter to intentionally halt the current execution context.
* **Pre-Scanned Function Entry:** Top-level executable code sitting outside a `mean` block is skipped by the interpreter pre-scan. The defined entry point is always `mean start() { ... } pass`.
* **Domain-Segregated Conditions:**
  * `if.condition(...)` handles logical evaluation (`:` is AND, `,` is OR, `?=?` is equality, with `+:`, `-:`, `*:`, `/:` for inline arithmetic).
  * `if.condition.math(...)` enforces strict numeric evaluation (`+`, `-`, `*`, `/`, `==`). Strings and booleans trigger an immediate `TypeCoercionError`.
* **Explicit Variable Allocation & Mutation (Option B):**
  * `dictate x = 10; pass` allocates a local variable.
  * `dictate.global g = "live"; pass` allocates a global variable.
  * `global(x).val = 20; pass` explicitly mutates an existing variable.
* **Scope-Tagged Interpolation:** String templates require explicit scope markers: `?[global(varName)]?` or `?[local(varName)]?`.
* **Stateful Generators:** Native primitives such as `bln.range([true: false: true])` yield successive states on each evaluation without external state tracking.
* **Deterministic Threading:** Line-tracked multithreading using `dictate dictate.<Name>.thread() { ... } pass`, synchronized via `.snap; pass` barriers and `.back(); pass` controller handshakes.

---

## Quick Syntax Cheat Sheet

```plaintext
// hello.dic
import.dictate("math_utils.dic"); pass

dictate dictate.Worker.thread() {
    dictate.global workerStatus = "Processing..."; pass
    .snap; pass
    global(workerStatus).val = "Complete"; pass
} pass

mean start() {
    // Variable allocation and array manipulation
    dictate items = ["alpha": "beta": "gamma"]; pass
    global(items).list.add("omega"); pass
    dictate firstItem = global(items).list.get(place.list = 0); pass

    // Scope-tagged template string
    console.dictate.global.In("Initial item: ?[local(firstItem)]?"); pass

    // Strict math evaluation
    dictate count = 10; pass
    if.condition.math(count + 5 == 15) {
        console.dictate.In("Math domain verified."); pass
    } pass

    // Thread launch and handshake
    dictate.global(dictate.Worker.thread()).open(); pass
    dictate.global(dictate.Worker.thread()).back(); pass
    console.dictate.global.In("Status: ?[global(workerStatus)]?"); pass
} pass
