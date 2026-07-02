# Usage & API

The public API lives at the module root (`github.com/go-ruby-rubocop/rubocop`). It
is **RuboCop-shaped but Go-idiomatic**: a `Cop` interface and an `Offense` model
mirror the gem's cop/offense framework, while the surface follows Go conventions —
value types, an explicit `error` on config parsing, no global state.

!!! success "Status: implemented"
    The library is built and importable as `github.com/go-ruby-rubocop/rubocop`,
    bound into `rbgo` as a native module; see [Roadmap](roadmap.md).

## Install

```sh
go get github.com/go-ruby-rubocop/rubocop
```

## Worked example — run the core cop set over a source

```go
package main

import (
	"fmt"

	"github.com/go-ruby-rubocop/rubocop"
)

func main() {
	src := "def foo(a, b)\n  return a + b\nend\n"

	// Parse an optional .rubocop.yml (empty => every cop at its default).
	cfg, _ := rubocop.ParseConfig("")

	// Run the built-in core cop set.
	run := rubocop.NewRunner(rubocop.DefaultRegistry(), cfg)
	offenses := run.Inspect("foo.rb", src)

	// Render like `rubocop --format simple`.
	report := rubocop.SimpleTextFormatter{}.Format([]rubocop.FileResult{
		{Path: "foo.rb", Offenses: offenses},
	})
	fmt.Print(report)
	// == foo.rb ==
	// C:  2:  3: [Correctable] Style/RedundantReturn: Redundant return detected.
	// W:  1: 12: [Correctable] Lint/UnusedMethodArgument: Unused method argument - b. ...
	//
	// 1 file inspected, 2 offenses detected, 2 offenses autocorrectable
}
```

## Parsing a `.rubocop.yml`

`ParseConfig` reads a `.rubocop.yml` (via [go-ruby-yaml](https://github.com/go-ruby-yaml/yaml))
into a `Config`: `AllCops.DisabledByDefault`, per-cop `Enabled`, and per-cop params
(`Max`, `EnforcedStyle`, …). A cop's effective config is its built-in default
merged with the file's overrides.

```go
cfg, err := rubocop.ParseConfig(`
AllCops:
  DisabledByDefault: false
Metrics/MethodLength:
  Max: 15
Style/StringLiterals:
  EnforcedStyle: single_quotes
`)
```

## Shape

```go
// A Cop inspects a lexed+parsed Source under its effective config and
// returns the offenses it found.
type Cop interface {
	Name() string
	Inspect(src *Source, cfg CopConfig) []Offense
}

// An Offense: what the gem reports, byte-for-byte.
type Offense struct {
	CopName     string   // e.g. "Style/RedundantReturn"
	Location    Location // 1-based line/column and source range
	Message     string   // gem-identical message text
	Severity    Severity // convention (C), warning (W), error (E), …
	Correctable bool
	Correction  Correction // source-range edit the host may apply
}

// The Registry holds the built-in cops; DefaultRegistry() returns the core set.
func DefaultRegistry() *Registry

// The Runner (the commissioner) runs every enabled cop over a Source.
func NewRunner(reg *Registry, cfg *Config) *Runner
func (r *Runner) Inspect(path, src string) []Offense

// Config model.
func ParseConfig(yaml string) (*Config, error)

// Formatters.
type SimpleTextFormatter struct{ /* ... */ }
type ProgressFormatter struct{ /* ... */ }
func (f SimpleTextFormatter) Format(results []FileResult) string
func (f ProgressFormatter) Format(results []FileResult) string
```

## Cops

The core set is 22 cops, detection + message + location validated against the gem:

| Department | Cops |
| ---------- | ---- |
| **Layout** | `TrailingWhitespace`, `TrailingEmptyLines`, `SpaceAfterComma`, `EmptyLines`, `IndentationWidth`, `LineLength` |
| **Style**  | `StringLiterals`, `FrozenStringLiteralComment`, `MethodDefParentheses`, `RedundantReturn`, `Not`, `IfUnlessModifier`, `GuardClause`, `NumericLiterals` |
| **Lint**   | `UselessAssignment`, `UnusedMethodArgument`, `DuplicateMethods`, `AmbiguousOperator`, `ShadowingOuterLocalVariable` |
| **Metrics**| `MethodLength`, `LineLength`, `ClassLength` (each with a configurable `Max`) |

## Gem conformance

Correctness is defined by the `rubocop` gem. A **differential oracle**
(version-gated on `RUBY_VERSION >= "4.0"`) runs a corpus of Ruby snippets through
both the gem and this library and compares the offenses (cop name + line/col +
message) plus the formatter output **byte-for-byte** — not approximated from
memory. The oracle skips itself where the gem (or a target-arch Ruby) is absent,
so the cross-arch lanes still validate the library against its deterministic
golden vectors.

## Relationship to Ruby

`go-ruby-rubocop/rubocop` is **standalone and reusable**, and is the RuboCop
backend bound into [go-embedded-ruby](https://github.com/go-embedded-ruby/ruby) by
`rbgo` as a native module — the same way
[go-ruby-regexp](https://github.com/go-ruby-regexp) and
[go-ruby-erb](https://github.com/go-ruby-erb) are bound. The dependency runs the
other way: this library has no dependency on the Ruby runtime.
