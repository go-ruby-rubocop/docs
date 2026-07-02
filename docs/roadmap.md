# Roadmap

`go-ruby-rubocop/rubocop` is grown **test-first**, each capability
differential-tested against the `rubocop` gem rather than built in isolation. The
deterministic, interpreter-independent slice of RuboCop — offense detection over a
Ruby source string — is **complete** for its core cop set.

| Stage | What | Status |
| --- | --- | --- |
| Cop framework & Offense model | A `Cop` interface (`Inspect(*Source, CopConfig) []Offense`), an `Offense{CopName, Location, Message, Severity, Correctable, Correction}` model, a `Registry` and a `Runner` (the commissioner) that runs every enabled cop over a lexed+parsed `Source` and returns offenses in RuboCop's report order. | **Done** |
| Core cop set (22 cops) | A faithful core set of 22 cops across Layout / Style / Lint / Metrics — detection, location and message text validated differentially against the gem. | **Done** |
| Configuration (.rubocop.yml) | `ParseConfig` reads a `.rubocop.yml` (via go-ruby-yaml) into a `Config`: `AllCops.DisabledByDefault`, per-cop `Enabled`, and per-cop params (`Max`, `EnforcedStyle`, …); a cop's effective config is its built-in default merged with the file's overrides. | **Done** |
| Formatters | `SimpleTextFormatter` (`--format simple`) and `ProgressFormatter` (`--format progress`), including the byte-faithful summary line (`N files inspected, M offenses detected[, K offenses autocorrectable]`). | **Done** |
| go-ruby-parser AST integration | Built on the go-ruby-parser Ruby AST and lexer; each cop inspects a Ruby source string through its lines, its token stream and its AST — no Ruby runtime — and computes a source-range `Correction` the host applies. | **Done** |
| Differential oracle & coverage | A corpus of Ruby snippets inspected both here and by the `rubocop` gem, cop name + line/col + message plus formatter output compared byte-for-byte; 100% coverage, gofmt + go vet clean, green across all six 64-bit Go arches and three OSes. | **Done** |

## Documented out-of-scope boundaries

These are **deliberate**, recorded so the module's surface is unambiguous:

- **A core cop set, not the whole gem.** RuboCop bundles ~500 cops; the ~480
  beyond the core set above are deliberately absent rather than stubbed.
  Parameterised cops honour the subset of gem params they document (`Max`,
  `Width`, `MinDigits`, `EnforcedStyle`, `MinBodyLength`); some non-`always`
  styles are recognised (they switch the cop off) but not yet re-implemented.
- **Detection here, application by the host.** Each cop computes the edit (a
  source-range `Correction`); rewriting the file is the consumer's job — that is
  why `rbgo` binds this module rather than the reverse.
- **No file-walking.** The library inspects a **source string**, not a directory
  tree. Locating files is the host's concern.
- **Reference is the `rubocop` gem.** Byte-for-byte conformance targets the gem's
  behaviour, matched to the version used by the differential oracle.
- **Standalone & reusable.** The module has no dependency on the Ruby runtime;
  the dependency runs the other way.

See [Usage & API](api.md) for the surface and [Why pure Go](why.md) for the
deterministic/interpreter split.
