# Why pure Go

`go-ruby-rubocop/rubocop` reimplements the core of Ruby's RuboCop linter in
**pure Go, with cgo disabled**. The slice of RuboCop it covers is
**deterministic and interpreter-independent**: detecting an offense from a source
string — its cop name, its line/column, its message text — is a pure function of
the source's lines, tokens and AST. No live binding, no evaluation of arbitrary
Ruby. That is exactly the part that can — and should — live as a standalone Go
library, separate from the interpreter.

## A core, built on go-ruby-parser

This library sits on the [go-ruby-parser](https://github.com/go-ruby-parser/parser)
Ruby AST and lexer. It is the RuboCop backend bound into
[go-embedded-ruby](https://github.com/go-embedded-ruby/ruby)'s `rbgo` as a native
module — the same pattern as [go-ruby-regexp](https://github.com/go-ruby-regexp)
and [go-ruby-erb](https://github.com/go-ruby-erb) — but a **standalone, reusable**
module so that:

- any Go program can import `github.com/go-ruby-rubocop/rubocop` directly, with no
  Ruby runtime;
- the dependency runs the *other* way — `rbgo` binds this module rather than this
  module depending on the interpreter;
- the behaviour is pinned by a **differential oracle** against the `rubocop` gem,
  independent of any one consumer.

## What it is — and isn't

Offense detection is deterministic and needs no interpreter, so it lives here as
pure Go. But the gem's file-walking and its ~500-cop breadth are out of scope:
this is a **core** cop set — 22 cops across Layout / Style / Lint / Metrics —
implemented faithfully rather than a fake of the whole. Applying autocorrections
is left to the host: each cop computes the edit (a source-range `Correction`) but
does not rewrite your file.

## Why pure Go matters here

Because the library is CGO-free, it:

- cross-compiles to every Go target with no C toolchain, and links into a single
  static binary;
- has **no dependency on the Ruby runtime** — the dependency runs the other way;
- can be differentially tested against the `rubocop` gem wherever one is present,
  while the cross-arch lanes (where the gem is absent) still validate the library
  against its deterministic golden vectors.

See [Usage & API](api.md) for the surface and [Roadmap](roadmap.md) for what is
in scope.
