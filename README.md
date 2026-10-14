# Honing-Dhar Stage 1 (scaffolding — NOT self-hosting yet)

**Stage 1 compiler-in-Dhar — scaffolding written in Dhar, assembled (3/7) by Stage 0 (`forge-of-dhar` HEAD 8326bf2). It does NOT self-host: Beta §4 gate `E1==E2` not attempted.**

Stage 0 is `src/lexer.asm` (~6731 lines at HEAD 8326bf2; 4798 at canon-baseline 465aa60), 100% NASM, Linux-only. All new work happens here, in the **verbose bootstrap subset only** (Strict Alpha §2.2 — no `x=5`, no `+=`, no `print`, no `forge{}` logic). See `PHASE0_BASELINE.md` (frozen + errata) for the true gate status.

## 1. Philosophy — Simple as Python, Fast as C (target, not status)

* **Verbose first:** Stage-1 source must itself be valid Stage-0 input: `flux x: i32 = 5`, `x = x + 1`, `task foo(a: i32, b: i32):`, `sys 1, 1, msg, len`. Concise forms (`x = 5`, `x += 1`, `foo(a,b):`, `print`) are **Stage-1 desugar work (Beta §2.1, Spec1 §4.3)** — designed here, not yet implemented, and correctly **rejected** by Stage 0 (`pure_concise.dhar` DHARC_FAIL is correct behavior).
* **Visual rule (design):** `{}` = `mold`/`forge` layout, `:` + indent = `task`/`when`/`span` logic. Stage 1 does not yet implement either side.
* **Zero-cost (design):** `mold`/`state`/`view` compile-time only; `choose` → `cmp`/`jne`, `each` → `peek` loop. No RA yet (`r12-r15` is a roadmap item, not code).

## 2. Language Reference (what Stage 1 source may use TODAY)

Strict Alpha §2.2 only — anything else fails the Stage-0 assembly gate and must not be in `src/*.dhar`:

| Category | Allowed in `src/*.dhar` today | NOT allowed (rejected by Stage 0) |
| :--- | :--- | :--- |
| Types | `i32`, `str`, `bool`, `raw[N]` (byte array) | `i8,i16,i64 u8,u16,u32,u64 f32,f64 char state`, generics, `option`, collections |
| Decls | `lock`/`flux` with `: TYPE`, `task`/`give`, `gen` | bare `x = 5`, `+=`, `print()`, `foo(a,b):` |
| Control | `when`/`shift`/`fallback`, `span`, `cycle`, `scan`, `escape`, `skip` | `choose`/`match`/`is`, `each`, `repeat`/`until`, `attempt`/`rescue` |
| Memory | `buf[i] = v` (byte), `peek`, `poke`, `sys`/`sysret` | `view`/`grab`, `mold` bodies, `forge{}`, member `p.x`, `text.len` |
| Modules | single-file + `task core():` | `pull`/`expose` (`todo`-gated in Stage 0) |

The old "doc-correct" tables (`i8..u64`, `f32/f64`, `view`/`grab`, `choose`/`match`/`each`, `mold Point{}`/`forge`, `pull math`, `state`/`trap`/`enforce`) describe **Spec1 targets for Beta**, not implemented code. They are kept in the Canon PDFs, not in this repo's build.

## 3. Concise Desugaring (backlog, NOT implemented)

| Concise (Beta §2.1) | Explicit (Alpha, what we write today) | Status |
| :--- | :--- | :--- |
| `x = 5` | `flux x: i32 = 5` | TODO — parser work |
| `x += 1` | `x = x + 1` (`inc` fold is Stage-0) | TODO |
| `print(msg)` | `sys 1,1,msg,len` | TODO |
| `foo(a,b):` | `task foo(a:i32,b:i32):` | TODO |
| `each y in buf` | `peek y, buf, idx` loop | TODO (Stage 2) |

`tests/pure_concise.dhar` (`y=10`, `y+=1`) documents the TODO: Stage 0 correctly rejects it.

## 4. Building (honest)

Via Stage 0, **from the Forge directory** (output is CWD-relative; the old vacuous-pass bug came from running with the wrong CWD — see `PHASE0_BASELINE.md` errata):

```bash
cd "../Forge of dhar (Stage 0)" && make linux
./build/dharc "../Honing Dhar (Stage 1)/src/main.dhar" --target=linux
# output is ./build/output.asm (in Forge) — assert it is FRESH before assembling:
ls -l --time-style=full-iso build/output.asm
nasm -f elf64 build/output.asm -o /tmp/m.o && ld -m elf_x86_64 -o /tmp/m /tmp/m.o
```

Per-module gate TODAY (re-run 2026-10-14): `ast/parser/symbol: ASM_OK (3/7)`; `main/codegen/lexer/compiler: NASM_FAIL or DHARC_FAIL (4/7)` — string-compare conditions, member access, bare-name immediates, `pull`. Assembly is not compilation: modules are stubs/emitters (`parse → give 0`, `symbol add → 0`, drivers print banners).

This repo's `Makefile` only assembles a pre-generated `build/output.asm` (`nasm → dhar_linux`); it does not invoke `dharc` and does not test. `build/` artifacts (`dhar1`, `test`, `output.asm`) are stale snapshots, not proof.

## 5. Pipeline (scaffolding, NOT working)

Claimed: `lexer` → `parser` → `symbol` → `codegen` → `main` → self-host.
Actual (`src/*.dhar`, 281 lines): `lexer.dhar` (49: keyword table + `give 0`), `parser.dhar` (9: `give 0`), `symbol.dhar` (15: `new → 0`), `codegen.dhar` (73: `sys`-based emitters only), `ast.dhar` (9: tags), `main.dhar` (79: reads a file, prints `[1/4]Lex…`, never calls pipeline), `compiler.dhar` (47: duplicate driver + 4 inert `pull`s, DHARC_FAILs). No `typecheck`/`ir` yet (canon target: 8 modules, `compiler.dhar` folds into `main.dhar`, all verbose-only).

## 6. Benchmark (Stage-0 numbers, NOT self-hosted)

There is no Stage-1-generated code to benchmark. `benchmarks/` mirrors Forge's suite (`orchestrator.py` real, `runners/*.sh` 0-byte stubs) and its numbers are **Stage-0 `dharc` emission**: V0.2.3 `Dhar 9.3 mean/8.2 med (~8.5x C)`, V0.3.0 `8.40/7.28 (~5.3x)`, σ≈mean (medians stable). The old `2.81ms (2.48x)` "Stage 1 via Stage 0" and "`all_new.dhar` self-hosts end-to-end" claims are withdrawn — no such file or measurement exists in this repo.

`benchmarks/workloads/test.dhar` (31-line triple-nested `span`) is a perf workload, not a correctness test. Correctness tests are `tests/` (10 files): `hello/test/rigorous/concise` PASS, `foolproof/reference/showcase` PASS after `state→st_val`, `pure_concise` CORRECT REJECT, `shift/loops` mirrors PASS via Stage 0 (4-branch, 5/12/5).

## 7. Roadmap (gated, not dated)

1. **Beta §3 source:** complete 8 verbose-only modules (`typecheck`, `ir` TO CREATE; fold `compiler.dhar`).
2. **Beta §4 proof:** `Alpha(S)→E1; E1(S)→E2; E1==E2` byte-identical. Entry criterion: every Spec1 construct used has a passing test through the exact pipeline; no banner-only stages.
3. **Then** Stage 2 (SSA, graph-coloring `r12-r15→r8-r15`, jump tables, `movdqu`), Stage 3 (`std`, `dhar build/test/fmt`). No performance or self-hosting claims until (2) passes.
