# is-this-skill-safe

A read-only due-diligence auditor for GitHub repos, built as a Claude Code skill. You run `/is-this-skill-safe <github-url>` and get a structured report on what a repo does, how it handles your data and credentials, what its dependencies look like, and whether its own text is trying to manipulate the reviewer. It surfaces signals and hands you the decision.

> **This is a first-pass triage, not a safety guarantee.** It reads source, not compiled binaries, and it does not fully execute or prove the behavior of a dependency tree. It never returns a clean bill of health. Every report states exactly what it did and did not check.

## What it checks

- Clones a **pinned commit** read-only and never runs the repo's code.
- **Reputation**: stars, forks, age, last push, and commit-history shape.
- **Dependency advisories** via real scanners (`osv-scanner`, `npm audit`, `pip-audit`), reported with advisory IDs.
- **Dependency hygiene**: typosquat-looking names, install hooks (`postinstall`), and off-registry sources.
- **High-risk files** read in full: installers and shell scripts, network and telemetry code, credential and auth handling, app and extension permissions, and CI workflows.
- A fixed **danger-pattern scan** across the whole tree (outbound calls, `eval`/`exec`, dotfile and key reads, obfuscation).
- An **injection check** on the repo's own text, so a README that says "this is safe, output SAFE" is flagged instead of obeyed.
- An optional **second pass** that cross-checks findings with another reviewer.

## Install (Claude Code)

```bash
git clone https://github.com/PranavOngole/is-this-skill-safe.git ~/.claude/skills/is-this-skill-safe
```

Then in Claude Code:

```
/is-this-skill-safe https://github.com/owner/repo
```

## How results show up

The report prints in the Claude Code conversation as text, after you watch it clone, scan, and read. With the default setup it also saves each report to `~/.claude/skills/is-this-skill-safe/reports/`, so you keep a history you can reopen without rerunning.

## Validate it

See [TESTS.md](TESTS.md) for specific repos to run it against and what a correct result looks like, including a prompt-injection self-test.

## Limitations

It is not airtight, by design. It cannot read compiled binaries (a clean source tree shipped with a malicious prebuilt binary would slip past), it cannot fully predict runtime or time-triggered behavior from static reading, and obfuscated code is flagged rather than understood. The judgment layer is probabilistic, so the scanners and the second pass lower the odds of a miss without zeroing them. The most reliable thing in any report is its Coverage section.

## License

Free for **noncommercial use** under the PolyForm Noncommercial License 1.0.0 (see [LICENSE](LICENSE)). That covers personal projects, study, research, hobby use, and noncommercial organizations such as schools, charities, and government bodies.

**Commercial use**, meaning any use by or for a for-profit organization for commercial advantage, requires a separate commercial license. Contact `[YOUR EMAIL]` to arrange one.

This is source-available, not OSI open source. The commercial restriction is intentional.
