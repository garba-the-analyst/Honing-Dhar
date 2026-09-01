# Honing-Dhar Stage 1 (V0.4.0-doc self-hosting)

**Stage 1 self-hosted compiler — written in Dhar, compiled by Stage 0 (`forge-of-dhar` V0.4.0-doc).**

Stage 0 is now V0.4.0-doc with doc-correct syntax.

## Syntax (per docs)

* **Types:** `i8,i16,i32,i64` `u8,u16,u32,u64` `f32,f64` `char` `str` `bool` `raw`
* **Vars:** `lock`/`flux`, **Mold:** `mold Name { }`, **Forge:** `forge Name { }`
* **Borrow:** `view`/`grab`, **Flow:** `when`/`fallback`/`shift`, `scan`/`span`/`cycle`
* **Project:** `pull`/`expose`, `core` entry
* **Error:** `state`/`trap`/`enforce` (data-driven, no unwinding)
* **Concise:** `x=5` → `flux x:i32`, `x+=1`, `foo(a,b):` inferred

No new keyword beyond docs.

## Build

```bash
../dhar-compiler/build/dharc src/main.dhar --target=linux
nasm -f elf64 build/output.asm -o build/main.o
ld -m elf_x86_64 -o build/main build/main.o
./build/main
```
