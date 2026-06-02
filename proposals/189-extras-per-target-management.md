# Feature Proposal: Per-Target Management for Extras

## Problem

When an extra has multiple targets, there is no first-class CLI way to add or remove a single target without recreating the whole extra.

Today a user who wants to add a second target to an existing extra must either:

- Run `skillshare extras init <name> --force` and re-list every target from scratch, or
- Hand-edit `config.yaml` directly.

Neither option is discoverable or scriptable. The same gap exists in the Web UI: the current `DELETE /api/extras` handler removes the whole extra, not a single target, so per-target management is missing on both surfaces.

## Proposed Solution

Add two new subcommands to `skillshare extras`:

```
skillshare extras add-target <name> --target <path> [--mode <mode>] [--flatten]
skillshare extras remove-target <name> --target <path>
```

Both commands support `--project`/`--global` mode flags and write to the oplog. `remove-target` intentionally does not delete already-synced files from disk — only the config entry is removed (same contract as `extras remove`).

Issue [#189](https://github.com/runkids/skillshare/issues/189). A reference implementation is available in PR [#188](https://github.com/runkids/skillshare/pull/188).

### Web UI parity

To stay consistent, the same capability should be available in the Web UI. This means extending the extras API with a per-target add/remove endpoint (e.g. `POST /api/extras/:name/targets` and `DELETE /api/extras/:name/targets/:path`) and surfacing it on the Extras page.

### Alternative: idempotent `extras init`

An alternative to two new subcommands is making `extras init <name> --target <new>` append a target when the extra already exists instead of failing or requiring `--force`. This keeps the surface area smaller at the cost of less explicit intent.

Both approaches are worth weighing. The new-verb approach makes intent clearer in scripts and help output; the idempotent-init approach avoids new subcommand names.

## Alternatives Considered

**Idempotent `extras init`** — append a target when the extra already exists. Simpler surface; less explicit. Discussed above.

**`extras edit` as a general mutation command** — a single `edit` subcommand that accepts `--add-target` and `--remove-target` flags. Possible, but the verb is ambiguous and harder to discover than `add-target`/`remove-target`.

**Hand-editing `config.yaml`** — works today but not scriptable or discoverable.

## Scope

- [x] Small (1-3 files, < 200 lines) — CLI only
- [ ] Medium (3-10 files, 200-500 lines) — CLI + Web UI API
- [ ] Large (10+ files, 500+ lines) — CLI + Web UI API + UI frontend

The CLI-only part is small. Adding Web UI parity (API handler + Extras page changes) brings it to medium.

## Open Questions

- Should `remove-target` warn or prompt when removing the last target from an extra (leaving it with zero targets)?
- Should `add-target` validate `--mode` against the extra's inherited default mode, or only against the explicitly provided `--mode` value?
- Prefer two new subcommands or idempotent `extras init`? (See Alternatives above.)
