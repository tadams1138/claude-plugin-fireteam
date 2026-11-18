# Fire Team

A Claude Code plugin that runs a gated, six-stage development pipeline for a single
feature: specification, test-first implementation, design review, refactoring,
specification reconciliation and a retrospective of the run itself.

Four agents, six stages. The orchestrator (the `/fireteam:fireteam` command) delegates
every stage, carries artifacts between them and stops at human gates.

| Stage | Agent | Produces |
|---|---|---|
| 1. Specification | `fireteam:fireteam-spec-author` | written specification and Gherkin feature files; proposed build/test commands and permission rules. No code of any kind. |
| 2. Implementation | `fireteam:fireteam-tdd-implementer` | step definitions, unit tests and production code, driven to green with the double loop of outer acceptance and inner TDD. |
| 3. Design review | `fireteam:fireteam-solid-reviewer` | a prioritized SOLID findings file in gitignored scratch. Read-only; no commit. |
| 4. Apply refactors | `fireteam:fireteam-tdd-implementer` | the findings the user accepted, applied under a green suite. |
| 5. Reconciliation | `fireteam:fireteam-spec-author` | the specification's status claims brought in line with what shipped. |
| 6. Debrief | `fireteam:fireteam-retro` | an audit of the run: boundary violations, instruction smuggling, rework loops, effort distribution. Changes nothing. |

Gates where the pipeline waits for you:

- **Toolchain approval** — the proposed `CLAUDE.md` build/test lines and permission rules.
- **Spec approval** — the scenarios, with the defaults for absent inputs and the
  counter-examples for every boundary surfaced by name.
- **Refactor selection** — which review findings to apply.

The implementation-review checkpoint after stage 2 passes automatically on a green suite
and escalates otherwise.

Every role is bound by `fireteam/constitution.md`: everything runs from the repository
root without `cd`, only the spec author touches the specification, refactors preserve
behavior, every stage hands off through a commit in a fixed format, and privileged files
are changed only with explicit human approval.

## Install

```
/plugin marketplace add tadams1138/claude-plugins
/plugin install fireteam@tadams-plugins
```

Then, in a git repository with a clean working tree:

```
/fireteam:fireteam <feature description>
```

## Requirements

- Git, and a bash: Linux, macOS, WSL, or Git Bash on Windows.
- The repository's build and test commands documented in its `CLAUDE.md`. If they are
  not, stage 1 proposes them for your approval.

## Contents

- `commands/fireteam.md` — the orchestrator.
- `agents/` — the four pipeline agents.
- `fireteam/constitution.md` — the rules every role follows.
- `fireteam/preflight.sh` — verifies and repairs the invariants the pipeline depends on.
- `fireteam/handoff.sh` — commits a stage in the constitutional handoff format.
- `fireteam/reviews.sh` — writes and manages review findings under `.claude/reviews/`.
- `fireteam/runlog.sh` — records stages and gates under `.claude/runs/` for the debrief.

Preflight adds `.claude/reviews/` and `.claude/runs/` to the repository's `.gitignore`.

## Permissions

The pipeline's own scripts run on every stage. To avoid a prompt each time, allow them
once in your user settings (`~/.claude/settings.json`). Replace `<plugin-root>` with the
plugin's installed directory, which is what `${CLAUDE_PLUGIN_ROOT}` resolves to; it
includes the plugin version, so update these rules after upgrading.

```json
"permissions": {
  "allow": [
    "Bash(<plugin-root>/fireteam/preflight.sh:*)",
    "Bash(bash <plugin-root>/fireteam/preflight.sh:*)",
    "Bash(<plugin-root>/fireteam/handoff.sh:*)",
    "Bash(bash <plugin-root>/fireteam/handoff.sh:*)",
    "Bash(<plugin-root>/fireteam/reviews.sh:*)",
    "Bash(bash <plugin-root>/fireteam/reviews.sh:*)",
    "Bash(<plugin-root>/fireteam/runlog.sh:*)",
    "Bash(bash <plugin-root>/fireteam/runlog.sh:*)"
  ],
  "deny": [
    "Bash(cd:*)",
    "Bash(pushd:*)"
  ]
}
```

The `bash`-prefixed entries cover Windows, where the scripts have no executable bit and
are invoked as `bash <script>`. Never allowlist writes to the plugin's `fireteam/`
directory or to `.claude/settings.json`; preflight fails if you do.

## License

MIT
