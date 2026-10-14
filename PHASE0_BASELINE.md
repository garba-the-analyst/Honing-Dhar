# Phase 0 Baseline — Honing Dhar (Stage 1)

 canon-baseline tag: 8fc2814 (commit), this file committed at 015473d.
 src/*.dhar: 279 lines total
   (lexer 49, parser 9, symbol 15, codegen 73, ast 9, main 78, compiler 46).
 Date: 2026-10-13. No compiler behavior change in Phase 0
 (commits since tag: main.dhar annotation cleanup, shipped-test rename).

## Assembly gate (via Stage-0 dharc at Forge 465aa60): 7/7 assemble

 lexer, parser, symbol, codegen, ast, main, compiler all exit 0 through
 dharc+NASM. Assembly is NOT self-hosting: modules are stubs/emitters
 (parser: give 0; symbol: give 0/-1; drivers print fixed banners and
 tok_count=len/4 without calling any pipeline task).

## Errata (found in Phase 1.5, recorded here; this file otherwise frozen)

 The 7/7 claim above is FALSE. The Phase-0 check ran dharc with CWD set so
 that output.asm landed in Honing/build while NASM assembled a STALE
 Forge/build/output.asm — every module "passed" vacuously. Re-ran
 correctly (dharc CWD=Forge) against the canon-baseline compiler:
   ast/parser/symbol: ASM_OK (true 3/7).
   main: NASM_FAIL at baseline (`mov [x], len` bare-name immediate —
     fixed by Forge Phase 1.2b identifier-initializer).
   codegen: NASM_FAIL at baseline (`text.len` member access — no member
     support; Phase 3 work, unchanged by Phase 1).
   lexer: NASM_FAIL at baseline (string-literal condition `word == "task"`
     emits `cmp rax, "task"` — string comparison unimplemented; Phase 3).
   compiler: NASM_FAIL at baseline (`len` bare-name, same root cause as
     main; now additionally DHARC_FAIL on `pull` per Phase 1.5c todo —
     slated for the Phase-3 fold into main.dhar).
 Lesson: gate scripts must assert on FRESH output paths (dharc unlinks
 stale output since Forge Phase 1.5b; verify CWD/build exists first).

## Shipped-test classification under Stage 0 (strict Alpha §2.2 subset)

 PASS (4): hello, test, rigorous, concise (declares flux x first: valid
   verbose assignment, not inference).
 REJECT (4): foolproof/reference/showcase pre-rename (state reservation;
   fixed by state->st_val rename, now compile+run correctly);
   pure_concise (bare y=10, y+=1: undeclared + += token). The pure_concise
   rejection is CORRECT Stage-0 behavior: concise inference/desugar is
   Stage-1 parser work (Beta §2.1, Spec1 §4.3), never Stage-0 input.

## Canon target (Beta §3): 8 modules — REACHED 2026-10-14 (mechanical pass)

 lexer, parser, ast, symbol, typecheck (CREATED, stub), ir (CREATED, stub),
 codegen, main. compiler.dhar (4 inert pull lines) DELETED 2026-10-14, folded
 into main.dhar per this target. All 8 are verbose-bootstrap-subset-only and
 assemble (see addendum below). Implementation (non-stub behavior) is still
 TODO throughout; assembly is NOT compilation (Beta §4 gate untouched).

## Self-host gate (Beta §4): not attempted in Phase 0

 S = complete 8-file source; Alpha(S)->E1; E1(S)->E2; E1==E2 byte-for-byte
 ignoring timestamps. Entry criterion: every Spec1 construct has a passing
 test through the exact pipeline; no progress-message-only stages.

## Addendum 2026-10-14 — gate re-run with fixed Stage-0 dharc + mechanical pass

 dharc = Forge main 4eda9a7 (owner-clobber fix). CWD=Forge, fresh output per
 module (the stale-output trap from the errata above was avoided):
   lexer/parser/symbol/ast/codegen/main/compiler(old 7):
     4/7 ASM-OK (parser/symbol/ast/main), lexer NASM-FAIL (`cmp rax,"task"`
     string-compare conditions), codegen NASM-FAIL (`text.len` member access),
     compiler DHARC-FAIL (`pull` todo — by design, then deleted).
   main.dhar healed vs baseline by Forge fixes (no Honing change).
 Mechanical pass (branch stage1/beta-source, this commit):
   lexer.dhar: 38 string-compare whens replaced by stub + TODO(Beta) — a `str`
     parameter cannot be indexed/compared in Strict Alpha, so the real matcher
     must scan raw bytes (design noted in-file). codegen.dhar: member access
     removed, lengths are explicit i32 params (one length miscount `14→13`
     fixed). compiler.dhar deleted. typecheck.dhar + ir.dhar created as
     assembling stubs.
   Result: 8/8 DHARC-OK + ASM-OK, 4/4 spot-checked RUN-OK (banners). All eight
   remain stubs/emitters — the Beta §4 proof has NOT advanced; only subset
   validity did. Stage-0 acceptance hole found during audit: `when x == "lit"`
   (string condition) is accepted but miscompiled instead of rejected —
   recorded as a Forge-side known limitation (outside-subset accept hole).

## Reproduce

 Run dharc with CWD=Forge (output is always CWD-relative
 CWD/build/output.asm); assert the file is fresh before assembling:

  cd [Forge] && ./build/dharc [Honing]/src/<module>.dhar --target=linux &&
  nasm -f elf64 build/output.asm -o /tmp/m.o (expect exit 0;
  proves assembly only, not compilation).
