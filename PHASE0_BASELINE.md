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

## Shipped-test classification under Stage 0 (strict Alpha §2.2 subset)

 PASS (4): hello, test, rigorous, concise (declares flux x first: valid
   verbose assignment, not inference).
 REJECT (4): foolproof/reference/showcase pre-rename (state reservation;
   fixed by state->st_val rename, now compile+run correctly);
   pure_concise (bare y=10, y+=1: undeclared + += token). The pure_concise
   rejection is CORRECT Stage-0 behavior: concise inference/desugar is
   Stage-1 parser work (Beta §2.1, Spec1 §4.3), never Stage-0 input.

## Canon target (Beta §3): 8 modules

 lexer, parser, ast, symbol, typecheck (TO CREATE), ir (TO CREATE),
 codegen, main. compiler.dhar (4 inert pull lines) folds into main.dhar.
 All 8 must be written ONLY in the verbose bootstrap subset above.

## Self-host gate (Beta §4): not attempted in Phase 0

 S = complete 8-file source; Alpha(S)->E1; E1(S)->E2; E1==E2 byte-for-byte
 ignoring timestamps. Entry criterion: every Spec1 construct has a passing
 test through the exact pipeline; no progress-message-only stages.

## Reproduce

 [Forge]/build/dharc src/<module>.dhar --target=linux &&
 nasm -f elf64 [Forge]/build/output.asm -o /tmp/m.o (expect exit 0;
 proves assembly only, not compilation).
