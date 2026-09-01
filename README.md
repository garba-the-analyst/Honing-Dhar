# Honing-Dhar Stage 1

**Stage 1 self-hosted compiler — written in Dhar, compiled by Stage 0 (`forge-of-dhar` V0.3.0).**

Stage 0 (`dhar-compiler`) is frozen and archived. All new work happens here.

## Concise Syntax Direction (V0.4.0)

Stage 1 keeps Stage 0's vocabulary (`task`, `when`, `span`, `flux`) but **requires fewer keywords** via inference — no new keywords needed yet:

| Verbose (Stage 0) | Concise (Stage 1, inferred) | How |
| :--- | :--- | :--- |
| `flux x: i32 = 5` | `x = 5` | Parser infers `flux` + `: i32` from `= 5` |
| `flux x: i32` | `x: i32` | `flux` inferred when `: type` present |
| `task foo(a: i32, b: i32):` | `foo(a, b):` | `task` + `: i32` inferred |
| `x = x + 1` | `x += 1` | Desugars to `x = x + 1` |
| `sys 1,1,msg,5` | `print(msg)` | Desugars to `sys` |
| `give x` | `x` (last expr) | Implicit return |

Both forms emit identical assembly. Stage 1's own source stays explicit (so Stage 0 can compile it); it *parses* concise for its inputs.

## New Keywords — Notification

No new keyword required for this stage. If needed, suggestions:
- `let` as alias for `flux` (more familiar than `flux`)
- `const` as alias for `lock`
- `loop` as alias for `span` (optional)

You will be notified before any keyword is added.

## Build (via Stage 0)

```bash
../dhar-compiler/build/dharc src/main.dhar --target=linux
nasm -f elf64 build/output.asm -o build/main.o
ld -m elf_x86_64 -o build/main build/main.o
./build/main
```

## Structure

- `src/lexer.dhar` — tokenizer (explicit + concise)
- `src/parser.dhar` — type inference, desugaring
- `src/ast.dhar` — AST nodes
- `src/symbol.dhar` — symbol table
- `src/codegen.dhar` — linear-scan RA (r12-r15)
- `src/main.dhar` — driver
