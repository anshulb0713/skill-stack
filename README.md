# skill-stack

A focused set of Claude Code skills + a persistent headless browser daemon. Seven specialist skills that cover the planning → review → QA loop, plus a fast browser tool the agent can drive end-to-end.

Works with **Claude Code** (primary) and **OpenAI Codex CLI**.

## Install

### Option 1 — paste this into Claude Code

Open Claude Code and paste the prompt below. Claude clones the repo, runs setup, and registers all skills.

> Install skill-stack:
> 1. Run `git clone --depth 1 https://github.com/anshulb0713/skill-stack.git ~/.claude/skills/skill-stack && cd ~/.claude/skills/skill-stack && ./setup`.
> 2. Verify the install by running `ls ~/.claude/skills/` — you should see individual directories for `office-hours`, `plan-eng-review`, `plan-design-review`, `review`, `design-shotgun`, `qa-only`, and `browse` alongside `skill-stack`. If you only see `skill-stack` and nothing else, setup did not finish — STOP and report the error from the previous step instead of continuing.
> 3. Only after the verify step passes, add a "skill-stack" section to CLAUDE.md that says to use the `/browse` skill for all web browsing and lists the available skills: `/office-hours`, `/plan-eng-review`, `/plan-design-review`, `/review`, `/design-shotgun`, `/qa-only`, `/browse`.
> 4. Tell me to restart Claude Code — the new slash commands only appear after a restart.

### Option 2 — run it yourself

**Requirements:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Git](https://git-scm.com/), [Bun](https://bun.sh/) v1.0+

```bash
git clone --depth 1 https://github.com/anshulb0713/skill-stack.git ~/.claude/skills/skill-stack
cd ~/.claude/skills/skill-stack
./setup
```

The setup script:
1. Installs Bun dependencies.
2. Compiles the `browse` browser daemon.
3. Symlinks (or copies, on Windows) each skill into `~/.claude/skills/` so Claude Code can discover them.

For **OpenAI Codex CLI** instead of Claude Code:

```bash
./setup --host codex
```

## Skills

| Skill | Role | What it does | Use it when... |
|-------|------|--------------|----------------|
| `/office-hours` | Product interrogator | Six forcing questions that expose demand reality, narrowest wedge, and future-fit. Also has a builder-mode for side-project brainstorming. | You have an idea and want to pressure-test it before writing code. |
| `/plan-eng-review` | Engineering manager | Locks in the execution plan — architecture, data flow, edge cases, test coverage. Interactive, opinionated. | You have a design doc and you're about to start coding. |
| `/plan-design-review` | Designer's eye | Rates each design dimension 0–10, explains what would make it a 10, then patches the plan to get there. | Your plan has UI/UX components and you want them reviewed before implementation. |
| `/design-shotgun` | Visual explorer | Generates multiple AI design variants, opens a comparison board, collects structured feedback. | You don't know what the UI should look like and want to see options. |
| `/review` | Pre-landing reviewer | Diff review against the base branch for SQL safety, LLM trust boundaries, conditional side effects, and structural issues. | You're about to land code changes and want a careful read first. |
| `/qa-only` | QA reporter | Systematically tests a web app and produces a structured report with health score, screenshots, and repro steps. Never edits code. | You want a bug report without any code changes. |
| `/browse` | Headless browser | Persistent Chromium daemon. Navigate, click, fill, screenshot, diff before/after, assert state. ~100ms per command after warm-up. | The agent needs to drive a browser — dogfooding, deploy verification, file uploads, authenticated flows. |

## How they fit together

`/office-hours` writes a design doc that `/plan-eng-review` and `/plan-design-review` lock down. `/design-shotgun` explores the visual direction in parallel. Once code is written, `/review` catches diff issues before merge and `/qa-only` reports browser-level bugs via `/browse`.

## Browse — quick reference

After install, the `$B` alias (or `browse` binary) runs against a long-lived daemon. First call cold-starts Chromium (~3s); subsequent calls are ~100–200ms.

```bash
$B goto https://example.com    # navigate
$B snapshot -i                  # see all interactive elements with refs
$B fill @e3 "hello"             # fill input by ref
$B click @e5                    # click button by ref
$B snapshot -D                  # diff vs. last snapshot — see what changed
$B screenshot /tmp/page.png     # capture
$B text                         # read page content
$B status                       # daemon health
$B stop                         # shut down daemon
```

State persists between calls — cookies, tabs, login sessions all carry over until the daemon idles out after 30 min.

## Project layout

```
skill-stack/
├── browse/              # browser daemon (Bun + Playwright + Chromium)
├── design/              # GPT Image API CLI (used by /design-shotgun)
├── hosts/               # host configs: claude.ts, codex.ts
├── scripts/             # build pipeline + skill-doc generator
├── bin/                 # CLI helpers
├── office-hours/        # ┐
├── plan-eng-review/     # │
├── plan-design-review/  # │
├── review/              # │ the 7 skills
├── design-shotgun/      # │
├── qa-only/             # │
└── browse/              # ┘ (browse skill points at the daemon above)
```

## Commands

```bash
bun install            # install dependencies
bun run gen:skill-docs # regenerate SKILL.md files from .tmpl templates
bun run build          # compile browse binary + generate skill docs
bun run dev <cmd>      # run browse CLI in dev mode without compiling
```
