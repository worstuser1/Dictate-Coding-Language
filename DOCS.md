```markdown
# Dictate Language — Complete Developer Reference (v1.1.0)

Dictate is an imperative, structured programming language built around explicit execution pacing (`pass`), strict logical and mathematical domain separation, namespace-routed state mutation, and deterministic concurrency.

---

## 1. File Structure, Pre-Scan, and Entry Point

* **File Extension:** All Dictate source files use the `.dic` extension.
* **Pre-Scan Phase:** Before running any code, the interpreter scans the entire `.dic` file to register `mean` subroutines, `.thread()` blocks, and top-level `import.dictate` modules.
* **Automatic Entry (`start`):** Normal execution begins exclusively at `mean start() { ... } pass`.
* **Orphaned Top-Level Code:** Any executable statement placed outside a `mean` or `.thread()` block is silently skipped during normal execution (and flagged as an *Outside Error* in Debug Mode).

```plaintext
// Ignored during Run Mode (Flagged as Outside Error in Debug Mode):
dictate orphanedVariable = 100; pass

// Program execution automatically begins here:
mean start() {
    console.dictate.In("Interpreter initialized inside start()."); pass
} pass

```

---

## 2. Comments

* **Single-Line Comments:** Begin with `//` and continue to the end of the line.
* **Multi-Line Comments:** Open with `/"` and close with `"/`, spanning across multiple lines.

```plaintext
// This is a single-line comment.

/"
   This is a multi-line comment.
   The pre-scanner strips it out before parsing statements.
"/
mean start() {
    console.dictate.In("Comments do not execute."); pass
} pass

```

---

## 3. Statement Termination (`;`) and Pacing (`pass`)

Dictate separates ending an instruction from advancing to the next line:

* **Semicolon (`;`):** Every statement must end with a semicolon. Omitting `;` triggers a parse-time `FatalSyntaxError`.
* **Advancement (`pass`):** `pass` is a structural directive that grants the interpreter permission to advance its line pointer to the next line. Every closing block brace also requires `pass` (`} pass`).
* **Intentional Stop (Omitting `pass`):** If a statement ends with `;` but omits `pass`, the interpreter treats that line as a deliberate stopping point and halts the current block immediately.

```plaintext
mean start() {
    console.dictate.In("Line 1 executes and advances."); pass
    console.dictate.In("Line 2 executes and intentionally halts the block here.");
    console.dictate.In("Line 3 is never reached because Line 2 omitted pass."); pass
} pass

```

---

## 4. Variables, Dynamic Types, Scope, and Mutation

Variables are dynamically typed based on their initial value (integers, floats, strings, booleans, arrays, maps, or generators). Dictate enforces a strict boundary between **allocating** a variable and **mutating** it:

* **Local Allocation (`dictate`):** Confined to the current block (`dictate name = value; pass`).
* **Global Allocation (`dictate.global`):** Visible across all functions and threads once executed (`dictate.global name = value; pass`). Redeclaring an existing global name raises a `StateResolutionError`.
* **Explicit Value Mutation (`global(...).val`):** Existing local or global variables cannot be overwritten with `dictate`. To update a variable's value after creation, use `global(identifier).val = newValue; pass`.

```plaintext
mean start() {
    // 1. Initial Declarations
    dictate retryCount = 1; pass
    dictate ratio = 3.14; pass
    dictate isReady = true; pass
    dictate.global serverStatus = "booting"; pass

    // 2. Explicit Mutations via .val
    global(retryCount).val = 2; pass
    global(serverStatus).val = "online"; pass

    console.dictate.In(retryCount); pass
} pass

```

---

## 5. Console Output, Scope-Tagged Interpolation, and Input

### Console Output

* `console.dictate(value); pass` prints inline without a trailing newline.
* `console.dictate.In(value); pass` prints the value followed by a newline.

### Scope-Tagged String Interpolation

* `console.dictate.global.In("..."); pass` evaluates inline variable tags inside strings:
* Global variables use `?[global(identifier)]?`.
* Local variables use `?[local(identifier)]?`.



### Typed Standard Input

* Read typed user input from the console via `input.dictate.str()`, `input.dictate.int()`, `input.dictate.flt()`, and `input.dictate.bln()`.

```plaintext
mean start() {
    dictate.global appName = "DictateCore"; pass
    dictate port = 8080; pass

    // Inline and newline output
    console.dictate("Status: "); pass
    console.dictate.In("Active"); pass

    // Scope-tagged interpolation for global and local variables
    console.dictate.global.In("Service ?[global(appName)]? running on port ?[local(port)]?."); pass
} pass

```

---

## 6. Arrays (Lists) and Indexed Operations

Arrays use square brackets with elements separated by **colons (`:`)**, not commas. All array operations route through the `global(arrayName).list` namespace:

* **Append to End:** `global(arrayName).list.add(value); pass`
* **Insert at Index:** `global(arrayName).list.add(value, index); pass`
* **Remove by Value:** `global(arrayName).list.sub(value); pass` (Raises `StateResolutionError` if the value is absent).
* **Remove by Index:** `global(arrayName).list.sub(place.list = index); pass`
* **Read by Index:** `global(arrayName).list.get(place.list = index)`

```plaintext
mean start() {
    // Colon-separated array literal
    dictate queue = ["init": "compile": "verify"]; pass

    // Append and positional insert
    global(queue).list.add("deploy"); pass
    global(queue).list.add("lint", 1); pass

    // Read element at index 0
    dictate firstStep = global(queue).list.get(place.list = 0); pass
    console.dictate.global.In("First step: ?[local(firstStep)]?"); pass

    // Remove by value and by index
    global(queue).list.sub("verify"); pass
    global(queue).list.sub(place.list = 0); pass

    console.dictate.In(queue); pass
} pass

```

---

## 7. Key-Value Maps (`dictate.map`)

Maps store key-value pairs using brace literals and mutate through `global(mapName).map`:

* **Declaration:** `dictate.map name = { "key": "value" }; pass`
* **Add / Update Key:** `global(mapName).map.add("key", value); pass`
* **Remove Key:** `global(mapName).map.sub("key"); pass` (Raises `StateResolutionError` if the key does not exist).

```plaintext
mean start() {
    dictate.map settings = { "theme": "dark", "timeout": 30 }; pass

    global(settings).map.add("retries", 5); pass
    global(settings).map.sub("theme"); pass

    console.dictate.In(settings); pass
} pass

```

---

## 8. Ranges, Randomization, and Stateful Generators

Dictate includes built-in sequence generators and string/math range utilities:

* **Random Integer (0 to 100):** `random.dictate()`
* **Bounded Random Integer:** `random.dictate(int.range(min, max))`
* **Range Midpoint:** `int.middle.range(start, end)`
* **String Length:** `str.range("text")`
* **Stateful Boolean Generator (`bln.range`):** When initialized with a colon-separated boolean list (`bln.range([true: false: true])`), every read of the variable automatically yields the next boolean state in the cycle. Passing numbers inside a `bln.range` list raises a `TypeCoercionError`.

```plaintext
mean start() {
    dictate roll = random.dictate(int.range(1, 20)); pass
    dictate midpoint = int.middle.range(10, 50); pass
    dictate nameLen = str.range("Dictate"); pass

    console.dictate.global.In("Roll: ?[local(roll)]? | Mid: ?[local(midpoint)]? | Len: ?[local(nameLen)]?"); pass

    // Stateful boolean toggle
    dictate toggle = bln.range([true: false]); pass
    console.dictate.In(toggle); pass // Outputs: true
    console.dictate.In(toggle); pass // Outputs: false
    console.dictate.In(toggle); pass // Outputs: true
} pass

```

---

## 9. Conditional Domains (`if.condition` vs. `if.condition.math`)

Dictate separates logical checks from strict mathematical checks. Chained comparisons (such as `10 < x < 20`) and nested `if.condition` headers are illegal and raise a `FatalSyntaxError`.

### 1. Logical Domain (`if.condition`)

* **AND Operator:** `:` (Higher precedence, short-circuits when left side is `false`).
* **OR Operator:** `,` (Lower precedence, short-circuits when left side is `true`).
* **Equality Operator:** `?=?`
* **Inline Arithmetic:** Must use colon-suffixed operators (`+:`, `-:`, `*:`, `/:`) so math symbols do not collide with logical delimiters. Bare arithmetic (`+`, `-`, `*`, `/`) inside `if.condition` raises a `FatalSyntaxError`.

### 2. Strict Math Domain (`if.condition.math`)

* Accepts standard arithmetic (`+`, `-`, `*`, `/`) and numeric comparisons (`==`, `!=`, `<`, `>`, `<=`, `>=`).
* Strictly forbids strings and booleans; evaluating non-numeric types immediately raises a runtime `TypeCoercionError`.

```plaintext
mean start() {
    dictate score = 75; pass
    dictate bonus = 10; pass
    dictate isMember = true; pass

    // Logical domain: '+:' for addition, ':' for AND, ',' for OR, '?='? for equality
    if.condition(score +: bonus > 80 : isMember ?=? true, score > 95) {
        console.dictate.In("Logical gate cleared."); pass
    } pass

    // Strict math domain: standard '+' and '==' allowed; strings/booleans forbidden
    if.condition.math(score + bonus == 85) {
        console.dictate.In("Strict math total verified."); pass
    } pass
} pass

```

---

## 10. Iteration (`for`) and Loop Breaking (`dictate.break;`)

* **`for` Loops:** Embed a `dictate` initializer, an `if.condition` or `if.condition.math` check, and a step expression (`i++` or `i--`), ending the block with `} pass`.
* **Breaking Loops (`dictate.break;`):** Immediately exits the enclosing loop and jumps past the loop's closing `} pass`.
* **The No-`pass` Rule on Break:** Never write `dictate.break; pass`. Because `dictate.break;` abandons the loop while `pass` tries to advance to the next line inside the loop, combining them raises a `FatalSyntaxError`.

```plaintext
mean start() {
    dictate maxCount = 6; pass

    for (dictate i = 0; if.condition(i < maxCount); i++) {
        if.condition(i ?=? 3) {
            console.dictate.In("Stopping loop early at index 3."); pass
            dictate.break;
        } pass
        console.dictate("Index: "); pass
        console.dictate.In(i); pass
    } pass
} pass

```

---

## 11. Subroutines, Parameters, and Return Values

User-defined subroutines are declared with `mean`, return values via `yield.dictate`, and are invoked through the `mean.dictate.<name>(...)` namespace:

```plaintext
mean calculateTax(dictate subtotal, dictate rate) {
    dictate tax = subtotal *: rate; pass
    yield.dictate tax; pass
} pass

mean printHeader() {
    console.dictate.In("=== Financial Summary ==="); pass
} pass

mean start() {
    // Call subroutine without return value
    mean.dictate.printHeader(); pass

    // Call function with arguments and capture yielded value
    dictate owed = mean.dictate.calculateTax(200, 2); pass
    console.dictate.global.In("Computed Value: ?[local(owed)]?"); pass
} pass

```

---

## 12. Concurrency and Line-Tracked Threading

Dictate provides line-tracked background threads with built-in barrier synchronization:

* **Thread Declaration:** `dictate dictate.<Name>.thread() { ... } pass`
* **Start Thread:** `dictate.global(dictate.<Name>.thread()).open(); pass`
* **Check Active State:** `dictate.global(dictate.<Name>.thread()).check()` (Returns `true` if running, `false` if stopped).
* **Track Completion Ratio:** `dictate.global(dictate.<Name>.thread()).placement()` (Returns a `0.0` to `1.0` float based on completed statements divided by total statements in the thread).
* **Force Completion:** `dictate.global(dictate.<Name>.thread()).return(); pass`
* **Barrier Handshake (`.snap` & `.back`):**
* `.snap; pass` (placed *inside* the thread) freezes the thread at that exact line.
* `dictate.global(dictate.<Name>.thread()).back(); pass` (called *outside* the thread) pauses the main program until the thread hits `.snap`, then releases both simultaneously. Calling `.back()` on a thread with no `.snap` raises a `StateResolutionError`.


* **Thread Monitoring Loop (`while.if.return.thread`):** Repeats while the thread is active. If `dictate.break;` is executed inside this loop, the main program automatically waits for the thread to finish before exiting the loop.

```plaintext
dictate dictate.FileIndexer.thread() {
    dictate.global syncMessage = "Phase 1: Scanning directories..."; pass
    .snap; pass
    global(syncMessage).val = "Phase 2: Index committed."; pass
} pass

mean start() {
    // 1. Launch thread
    dictate.global(dictate.FileIndexer.thread()).open(); pass

    // 2. Wait for thread to hit .snap, then resume both
    dictate.global(dictate.FileIndexer.thread()).back(); pass
    console.dictate.global.In("Handshake synced: ?[global(syncMessage)]?"); pass

    // 3. Monitor completion percentage until finished
    dictate.global(dictate.FileIndexer.thread()).while.if.return.thread {
        dictate pct = dictate.global(dictate.FileIndexer.thread()).placement(); pass
        console.dictate("Thread Progress: "); pass
        console.dictate.In(pct); pass

        if.condition.math(pct >= 0.5) {
            dictate.break;
        } pass
    } pass

    console.dictate.global.In("Final State: ?[global(syncMessage)]?"); pass
} pass

```

---

## 13. Multi-File Modules and Scoped JSON I/O

### 1. Cross-File Module Imports (`.dic`)

Place `import.dictate("filename.dic"); pass` at the top of a file before `mean start()`. It merges all `mean` subroutines and `.thread()` blocks from the external file while ignoring any orphaned top-level code inside that module.

### 2. Scoped JSON Reading (`.open(r)` & `.load()`)

Open a JSON file in a read-only block using `import.dictate("file.json").open(r) { ... } pass` and parse its contents into a variable using `import.dictate.json.load()`.

### 3. JSON Exporting (`.export`)

* **Overwrite Mode (`rw`):** `import.dictate.json.export("file.json", variableName, rw); pass` replaces the file with the serialized map or list.
* **Append Mode (2 arguments):** `import.dictate.json.export("file.json", variableName); pass` merges or appends the variable into the existing JSON file.

```plaintext
//Top-level module import
import.dictate("math_utils.dic"); pass

mean start() {
    // Read JSON inside a scoped handle block
    import.dictate("config.json").open(r) {
        dictate configData = import.dictate.json.load(); pass
        console.dictate.In(configData); pass
    } pass

    // Persist a new map to output.json (overwrite mode)
    dictate.map report = { "status": "ok", "code": 200 }; pass
    import.dictate.json.export("output.json", report, rw); pass
} pass

```

---

## 14. Structured Error Handling (`try.dictate` / `trap.condition`)

Wrap fallible runtime operations inside `try.dictate { ... } trap.condition(err) { ... } pass` to catch runtime exceptions (`StateResolutionError`, `TypeCoercionError`, or division by zero) without crashing the interpreter:

```plaintext
mean start() {
    try.dictate {
        dictate badMath = 10 /: 0; pass
    } trap.condition(err) {
        console.dictate("Recovered from runtime error: "); pass
        console.dictate.In(err); pass
    } pass
} pass

```

### Error Classification Reference

| Error Class | Phase | Trigger Conditions |
| --- | --- | --- |
| **`FatalSyntaxError`** | Parse-Time | Missing `;`, using `dictate.break; pass`, chained comparisons (`a < b < c`), nested conditions in headers, or bare math inside `if.condition`. |
| **`TypeCoercionError`** | Runtime | Passing strings/booleans into `if.condition.math`, or placing numeric values inside a `bln.range` list. |
| **`StateResolutionError`** | Runtime | Redeclaring a `dictate` or `dictate.global` variable, removing a missing list/map item, out-of-bounds index access, missing `.snap` on `.back()`, or missing files. |
| **`SilentLogicError`** | Pre-Scan / Pacing | Placing orphaned code outside a `mean` block, or accidentally omitting `pass` after `;` or `}` (audited via **Debug Mode**). |

```

---

### How This Connects to Your Source Spec
* **Core V Foundations:** Preserves the `.dic` file rules, `mean start()` entry gate, `;` and `pass` pacing, `//` and `/" ... "/` comments, `dictate` vs. `dictate.global` scopes, colon-separated arrays, `bln.range` generators, `if.condition` vs. `if.condition.math` domains, and the four-tier error taxonomy[cite: 1, 2, 3, 4, 5, 6, 7, 9, 13, 14, 16].
* **Implemented Extensions:** Formalizes maps (`dictate.map`), parameterized functions (`yield.dictate`), error trapping (`try.dictate` / `trap.condition`), cross-file module imports (`import.dictate`), and scoped JSON read/export blocks[cite: 20, 21, 22, 23, 25], alongside your Option B `.val` / `.get` accessors and `.thread()` synchronization primitives.

<ElicitationsGroup>
</ElicitationsGroup>

```
