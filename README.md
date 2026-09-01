# Honing-Dhar Stage 1

**Stage 1 self-hosted compiler — written in Dhar, compiled by Stage 0 (`forge-of-dhar` V0.3.0).**

Stage 0 (`dhar-compiler`) is frozen and archived. All new work happens here.

## Concise Syntax (V0.4.0)

Stage 1 keeps Stage 0's vocabulary but **requires fewer keywords** via inference — no new keywords beyond the 13 already wired:

| Verbose (Stage 0) | Concise (Stage 1) | Desugaring |
| :--- | :--- | :--- |
| `flux x: i32 = 5` | `x = 5` | `flux x:i32 = 5` inferred |
| `x = x + 1` | `x += 1` | `x = x + 1` |
| `sys 1,1,msg,5` | `print(msg)` | `sys` |
| `task foo(a: i32):` | `foo(a, b):` | `task foo(a:i32,b:i32):` |

Both emit identical `r12-r15` RA code.

## New Control Flow (V0.5.0) — 13 new keywords wired

All nest via indentation:

* **switch/case:** `choose x:` → `is 1:` → `fallback:`  (codegen: `cmp r12,1` / `jne`)
* **match:** `match x:` → `is 1:` (pattern)
* **break/continue:** `escape` / `skip` (jump to loop end/start)
* **do-while:** `repeat:` → `until x < 10:`
* **for-each:** `each y in buf:` (declares `y`)
* **exceptions:** `attempt:` → `rescue e:` → `ensure:` + `raise 42` (landing pads)

No new keyword will be added without notification.

## Build (via Stage 0)

```bash
../dhar-compiler/build/dharc src/main.dhar --target=linux
nasm -f elf64 build/output.asm -o build/main.o
ld -m elf_x86_64 -o build/main build/main.o
./build/main
```

## Structure

- `src/lexer.dhar` — file I/O + concise `x=5`, `+=` tokenization
- `src/parser.dhar` — `x=5` → `flux`, `choose`/`attempt` desugaring
- `src/ast.dhar` — AST nodes
- `src/symbol.dhar` — symbol table
- `src/codegen.dhar` — linear-scan `r12-r15` + `choose`/`attempt` pads
- `src/main.dhar` — driver (read → lex → parse → emit)
- `src/compiler.dhar` — full pipeline
