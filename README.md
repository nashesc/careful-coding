# careful-coding

A Claude Code / Claude.ai skill that reduces common LLM coding mistakes: silently guessing at ambiguous requirements, overengineering simple requests, drive-by refactoring code nobody asked you to touch, and treating vague instructions as if they were verifiable done-conditions.

Based on [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on recurring LLM coding failure modes, repackaged as a Claude **skill** (not a CLAUDE.md drop-in) so it triggers automatically on non-trivial coding tasks instead of needing to be pasted into every project.

## What it does

Four checks, summarized in `SKILL.md` and run automatically when Claude takes on non-trivial coding work:

| Principle | Stops Claude from... |
|---|---|
| **Think Before Coding** | Silently picking one interpretation of an ambiguous request and running with it |
| **Simplicity First** | Building abstractions, config options, or error handling nobody asked for |
| **Surgical Changes** | "Improving" adjacent code, reformatting, or refactoring while fixing something unrelated |
| **Goal-Driven Execution** | Treating "make it work" as done without a verifiable test or check |

Full before/after code examples for each principle live in `references/examples.md` — Claude loads that file on demand when it needs to show a detailed diff, not on every trigger (keeps the always-loaded SKILL.md short).

## Install

### Claude Code (project-level)
Copy the folder into your project's skills directory:
```bash
cp -r careful-coding /path/to/your-project/.claude/skills/
```

### Claude Code (user-level, all projects)
```bash
cp -r careful-coding ~/.claude/skills/
```

### Claude.ai
Upload the `.skill` package (or the folder, if your client supports folder upload) via Settings → Skills.

## How it triggers

This is a skill, not a system prompt — Claude decides whether to consult it based on the `description` field in `SKILL.md`'s frontmatter, matched against your request. It's scoped to **non-trivial** coding tasks (new features, bug fixes, refactors, anything with more than one reasonable approach or unclear scope). It deliberately does not fire on trivial one-liners or obvious typo fixes — see the Tradeoff note in `SKILL.md`.

If it's under- or over-triggering for your workflow, the fix is editing the `description` field, not the body — that's the only field Claude uses to decide whether to load the skill at all.

## Structure

```
careful-coding/
├── SKILL.md              # Always loaded when the skill triggers (~80 lines)
└── references/
    └── examples.md        # Full before/after code walkthroughs, loaded on demand
```

This follows Claude's skill progressive-disclosure convention: keep the always-loaded body short, push detailed/long-form material into `references/` that's read only when actually needed.

## Customizing

- **Scope creep / wrong triggering**: edit the `description` line in `SKILL.md`'s frontmatter — that's the entire trigger mechanism.
- **Conflicts with other installed skills**: if you run another skill that mandates a specific verbose output format (e.g. an architecture-brief-first workflow), check it against this skill's "Simplicity First" and "Think Before Coding" principles before running both — they can give Claude contradictory instructions about whether to produce upfront structure.
- **Adding examples**: append to `references/examples.md` following the existing pattern (❌ wrong / ✅ right code blocks per principle); keep SKILL.md's condensed inline examples short, the long-form ones belong in the reference file.

## Tradeoff

These checks bias toward caution over speed. For genuinely trivial tasks, Claude is instructed to use judgment rather than apply all four ceremonially. If you find it slowing down simple work, that's a sign the description needs tightening, not that the principles are wrong.