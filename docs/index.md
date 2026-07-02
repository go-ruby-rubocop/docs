# go-ruby-rubocop documentation

**The core of Ruby's RuboCop linter in pure Go — cops, offenses, `.rubocop.yml` and formatters, no cgo.**

`go-ruby-rubocop/rubocop` is a faithful, pure-Go (zero cgo) reimplementation of the
core of Ruby's RuboCop linter, matching the `rubocop` gem's offenses byte-for-byte.
The module path is `github.com/go-ruby-rubocop/rubocop`.

It is built on the [go-ruby-parser](https://github.com/go-ruby-parser/parser) Ruby
AST and lexer: it inspects a Ruby **source string** — its lines, its token stream
and its AST — and emits offenses whose cop name, location and message match those
the gem produces, **without any Ruby runtime**. It is the RuboCop backend bound
into [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) by `rbgo` as a
native module — just like [go-ruby-regexp](https://github.com/go-ruby-regexp) and
[go-ruby-erb](https://github.com/go-ruby-erb) — but a **standalone, reusable**
module: the dependency runs the other way, with no dependency on the Ruby runtime.

!!! success "Status: framework + core cop set complete — gem byte-exact"
    The offense/cop framework (`Cop` interface, `Offense` model, `Registry`, `Runner`/commissioner), a **core cop set of 22 cops** across **Layout / Style / Lint / Metrics**, the **`.rubocop.yml`** config model (`ParseConfig` via go-ruby-yaml) and the default formatters (`SimpleTextFormatter`, `ProgressFormatter` with the byte-faithful summary line). Each cop computes a source-range `Correction`; the host applies the edit. Validated by a **differential oracle** against the `rubocop` gem — cop name, location and message compared byte-for-byte — at 100% coverage, `gofmt` + `go vet` clean, CI green across the six 64-bit Go targets and three OSes.

## Quick taste

```go
src := "def foo(a, b)\n  return a + b\nend\n"

cfg, _ := rubocop.ParseConfig("")                 // empty => every cop at its default
run := rubocop.NewRunner(rubocop.DefaultRegistry(), cfg)
offenses := run.Inspect("foo.rb", src)            // []Offense in RuboCop report order

report := rubocop.SimpleTextFormatter{}.Format([]rubocop.FileResult{
    {Path: "foo.rb", Offenses: offenses},
})
// C:  2:  3: [Correctable] Style/RedundantReturn: Redundant return detected.
// W:  1: 12: [Correctable] Lint/UnusedMethodArgument: Unused method argument - b. ...
// 1 file inspected, 2 offenses detected, 2 offenses autocorrectable
```

## Repositories

| Repo | What it is |
| --- | --- |
| [`rubocop`](https://github.com/go-ruby-rubocop/rubocop) | the library — Ruby's RuboCop linter core in pure Go |
| [`docs`](https://github.com/go-ruby-rubocop/docs) | this documentation site (MkDocs Material, versioned with mike) |
| [`go-ruby-rubocop.github.io`](https://github.com/go-ruby-rubocop/go-ruby-rubocop.github.io) | the organization landing page (Hugo) |
| [`brand`](https://github.com/go-ruby-rubocop/brand) | logo and brand assets |

## Principles

- **Pure Go, `CGO_ENABLED=0`** — trivial cross-compilation, a single static
  binary, no C toolchain.
- **AST-based, no interpreter.** Detecting an offense from a source string is
  deterministic and needs no Ruby runtime; it runs over the go-ruby-parser AST
  and lexer.
- **Gem byte-exact.** Cop name, location and message text match the `rubocop`
  gem exactly, not approximately, validated by a differential oracle against the
  gem.
- **Standalone & reusable.** No dependency on the Ruby runtime — the dependency
  runs the other way.
- **100% test coverage** is the target, enforced as a CI gate, across 6 arches
  and 3 OSes.

## Where to go next

- [Why pure Go](why.md) — why offense detection is deterministic enough to live
  as a standalone, interpreter-independent Go library.
- [Usage & API](api.md) — the public surface (`Cop`, `Offense`, `Registry`,
  `Runner`, `ParseConfig`, the formatters) and worked examples.
- [Roadmap](roadmap.md) — what is done and what is out of scope by design.

Source lives at [github.com/go-ruby-rubocop/rubocop](https://github.com/go-ruby-rubocop/rubocop).
