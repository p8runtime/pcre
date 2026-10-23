# p8-pcre

PCRE2's regular expressions in safe Rust: no C, no `unsafe`, no JIT.

p8 (`p8runtime/runtime`) uses it for PHP's `preg_*` functions (APS 6), and it is published on crates.io as `p8-pcre` for anyone who needs PCRE semantics without linking C.

**Status:** planned, not started. The runtime comes first.

## Plan

- **Port PCRE2 10.44's compiler and backtracking matcher.** That's about 22k lines of C, or roughly 15–20k lines of Rust. 10.44 is the version php-src 8.4 bundles.
- **Generate the Unicode tables** from Unicode's data rather than porting them.
- **Leave out:**
  - the JIT (it emits machine code, so it would need `unsafe`);
  - the DFA matcher, pattern conversion and the debug printer, which PHP doesn't use.
- **Make it fast without a JIT:** patterns that don't need backtracking (no backreferences, lookaround, recursion or backtracking verbs) run on the `regex` crate's automata. Only the rest use the backtracking matcher.
- **Match PCRE2 exactly:** match results, capture groups, `(*MARK)`, error messages and their offsets, and match and depth limits.

## Gates

- PCRE2's own test suite (`testdata/testinput*`, about 1.8 MB of expected output), driven by a `pcre2test`-compatible runner.
- php-src's `ext/pcre` tests, through p8.
- `#![forbid(unsafe_code)]` and no C in the dependency tree.

## Why not wrap PCRE2

Every PCRE crate on crates.io wraps the C library. p8 builds no C (p8 APS 2).

The `regex` crate deliberately leaves out backtracking features, and `fancy-regex` covers only part of them. This crate fills that gap.
