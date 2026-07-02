# Performance

`go-ruby-rubocop/rubocop` is the pure-Go library that
[`rbgo`](https://github.com/go-embedded-ruby/ruby) binds for Ruby's RuboCop core.
This page records the **methodology** for a comparative benchmark of that module
against the reference Ruby runtimes, part of the ecosystem-wide per-module parity
suite.

## What is measured

The **same** Ruby script — running the core cop set over a representative Ruby
source and formatting the offenses — is run under every runtime. `rbgo`'s number
reflects **this pure-Go library doing the work**; every other column is that
interpreter running the `rubocop` gem. So the comparison is the **Ruby-visible
operation**, apples-to-apples across interpreters. The script prints a
deterministic checksum of the offenses and its output is checked **byte-identical
to the gem** before timing.

- **Method:** best-of-5 wall time (best, not mean, to suppress scheduler noise);
  single-shot processes, no warm-up beyond the script's own loop.
- **Runtimes:** MRI (the oracle) and MRI `--yjit`; `jruby` (OpenJDK);
  `truffleruby` (GraalVM CE Native).
- The benchmark script and harness live in rbgo's repo under
  [`bench/modules/`](https://github.com/go-embedded-ruby/ruby/tree/main/bench/modules)
  (`rubocop.rb` + `run.sh`). Reproduce:
  `RBGO=./rbgo TRUFFLE=truffleruby bash bench/modules/run.sh 5`.

## Result

!!! note "Measurement pending"
    Numbers for this module are **not yet published**. The methodology above is
    fixed, but the comparative table is deliberately left empty rather than filled
    with fabricated figures — in line with the ecosystem convention of publishing
    only real, reproducible measurements. Once the parity run has been executed on
    a pinned host, the best-of-5 wall times (and the `vs gem` ratios) will be
    recorded here.
