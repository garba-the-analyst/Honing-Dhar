# Honing-Dhar Stage 1 (V0.4.0-doc self-hosting)

**Stage 1 self-hosted compiler — written in Dhar, compiled by Stage 0 (`forge-of-dhar` V0.4.0-doc).**

Stage 0 (`dhar-compiler`, NASM, 4153 lines) is frozen and archived. All new work happens here. Stage 1 desugars concise syntax and adds `mold`/`shift`/`state` before emitting the same lean `x86_64` with `r12-r15` RA.

## 1. Philosophy — Simple as Python, Fast as C

* **Fewer keywords via inference:** `x = 5` → `flux x: i32 = 5`, `x += 1` → `x = x + 1`, `foo(a,b):` → `task foo(a:i32,b:i32):`, `print(msg)` → `sys 1,1,msg,len`. No new runtime.
* **Visual rule:** `{}` = `mold`/`forge` (memory layout), `:` + indent = `task`/`when`/`span` (logic). Instantly distinct.
* **Zero-cost:** `mold`/`state`/`view` are compile-time only; `choose` → `cmp`/`jne`, `each` → `peek` loop, all `r12-r15`.

## 2. Language Reference

### Data Types (doc-correct)

| Category | Types |
| :--- | :--- |
| Integers | `i8,i16,i32,i64` `u8,u16,u32,u64` |
| Floats | `f32,f64` |
| Text | `char` `str` `bool` |
| Memory | `raw[N]` (`*mut`/`*const`), `view`/`grab` borrows |

### Variables

| Verbose | Concise | Inferred |
| :--- | :--- | :--- |
| `flux x: i32 = 5` | `x = 5` | `flux` + `:i32` |
| `flux x: i32` | `x: i32` | `flux` |
| `lock y: i32 = 1` | `y = 1` (top-level `lock` still explicit for globals) | `lock` when `lock` present |

### Control Flow

| Concept | Syntax | Notes |
| :--- | :--- | :--- |
| `if` / `else if` / `else` | `when` / `shift` / `fallback` | `shift` replaces `else if` |
| `while` | `span cond:` | check before |
| `for` | `scan x in buf:` | declares `x`, `buf` is `raw` |
| `loop` | `cycle:` | infinite, `escape` to break |
| `switch` | `choose x:` → `is 1:` / `fallback:` | `is` compares `choose` subject |
| `match` | `match x:` → `is 1:` | pattern, `is y:` binds |
| `break`/`continue` | `escape` / `skip` | innermost `span`/`scan`/`cycle`/`repeat` |
| `do-while` | `repeat:` → `until cond:` | check after |
| `for-each` | `each y in buf:` | `peek y, buf, idx` loop |

### Data Layout vs Logic

```dhar
mold Point {           # {} = memory layout
    flux x: i32
    flux y: i32
}
forge Point {          # {} = attach methods
    task init(a: i32, b: i32):
        x = a
        y = b
}
task dist(p: view Point):  # view = & (read), grab = &mut
    give p.x + p.y
```

### Project Structure

No `import`/`include`. Use `pull` / `expose`, entry is `task core():`:

```dhar
pull math
expose dist
task core():
    flux p = Point(1,2)
```

### Error Model (data-driven, no unwinding)

```dhar
flux s: state = may_fail()
trap s               # handle Error
enforce s == 0       # panic if critical
```

* `state` = `Success|Error`, `trap` handles, `enforce` panics. No stack unwinding.

## 3. Concise Desugaring (no new keyword)

| Concise | Explicit |
| :--- | :--- |
| `x = 5` | `flux x: i32 = 5` |
| `x += 1` | `x = x + 1` → `inc r12` |
| `print(msg)` | `sys 1,1,msg,len` |
| `foo(a,b):` | `task foo(a:i32,b:i32):` |
| `y = 10` in `each` | `flux y: i32` + `peek` |

## 4. Building

Via Stage 0:

```bash
../dhar-compiler/build/dharc src/main.dhar --target=linux
nasm -f elf64 build/output.asm -o build/main.o
ld -m elf_x86_64 -o build/main build/main.o
./build/main > /tmp/out.asm && nasm -f elf64 /tmp/out.asm -o /tmp/out.o && ld -m elf_x86_64 -o /tmp/out /tmp/out.o && /tmp/out
```

Each `src/*.dhar` compiles alone for testing.

## 5. Pipeline (6 tasks)

`lexer` (file I/O `x=5`/`+=`) → `parser` (`x=5`→`flux`) → `symbol` (`view`/`grab`) → `codegen` (`r12-r15`, `choose`→`cmp`/`jne`, `attempt`→landing pads, `each`→`peek`) → `main` (read→lex→parse→emit) → self-host (`pure_concise.dhar` → `mov r12,10` valid).

All 7 `src/*.dhar` compile via Stage 0.

## 6. Benchmark

Stage 1 output (via Stage 0) on `span i < 1M` loop: **2.81ms** vs C `1.13ms` (2.48×). Previous V0.3 string loop: `8.5×`. Full `all_new.dhar` (choose + `each` file + `attempt`) self-hosts end-to-end.

## 7. Roadmap

Stage 1 v1.0-honing is tagged. Stage 2: SSA, graph-coloring `r12-r15` → `r8-r15`, `choose` jump tables, `each` `movdqu` vectorization. Stage 3: `std` + `dhar build/test/fmt` for masses.
