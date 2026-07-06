# Universal Prompt — Generate a project's `.claude/` harness

> Copy-paste this prompt into Claude Code (VS Code extension or CLI) at the root
> of **any** project, in **any** language, of **any** scale. Claude will detect
> your stack, propose a plan (you validate), then generate the full harness.

---

```markdown
# MISSION: Generate this project's `.claude/` harness

You are an architect specialized in context engineering for coding agents.
Your mission: create a complete `.claude/` structure, tailored to THIS project,
strictly applying AI-guided development harness best practices (cost control +
reliability).

Work in 3 SEQUENTIAL PHASES. DO NOT move to the next phase without my explicit
approval.

═══════════════════════════════════════════════════════════════════
PHASE 1 — ANALYSIS (read-only, be economical)
═══════════════════════════════════════════════════════════════════
Explore the project WITHOUT writing anything. Use Glob/Grep and targeted reads
(no unnecessary full files, never lockfiles/dist/node_modules).
Detect and report CONCISELY:
- Main language(s) and framework(s)
- Package manager + REAL commands (install, dev, build, test, lint, typecheck)
- Test runner and test-naming convention
- Folder structure (where source code, tests, and config live)
- Visible conventions (lint/format, commit style, architectural patterns)
- Sensitive zones (auth, secrets/.env, migrations, infra/IaC, payments…)
- Project scale (single-file, small, medium, large, monorepo)
- Any existing `.claude/` or `CLAUDE.md`

End the phase with: "ANALYSIS COMPLETE — proposed plan below" followed by the plan.

═══════════════════════════════════════════════════════════════════
PHASE 2 — PLAN (validation required)
═══════════════════════════════════════════════════════════════════
Propose the `.claude/` structure you will create, ADAPTED to the detected scale:
- Small project → minimal harness (CLAUDE.md + settings + 1-2 agents + session_state).
- Medium/large project → full harness (agents, commands, skills, MCP).
List each file precisely + its role in one line.
State which agents/skills are relevant FOR THIS PROJECT (do not create a useless
skill). End with: "PLAN READY — validate before generation."
STOP and wait for my approval.

═══════════════════════════════════════════════════════════════════
PHASE 3 — GENERATION (only after approval)
═══════════════════════════════════════════════════════════════════
Create the files, strictly respecting these standards:

TARGET STRUCTURE (adapt to the approved plan):
.
├── CLAUDE.md
├── session_state.md
├── .mcp.json                      (if relevant)
└── .claude/
    ├── settings.json
    ├── agents/
    │   ├── architect.md           (Plan: most capable model, read-only)
    │   ├── codebase-explorer.md   (Explore: fast/cheap model, read-only)
    │   ├── code-reviewer.md        (Review: fast model, read-only)
    │   └── test-runner.md          (Verify: fast model, bounded maxTurns)
    ├── commands/
    │   └── review-pr.md
    └── skills/                     (only if a real need exists)
        └── <project-specific-skill>/
            ├── SKILL.md
            ├── scripts/            (deterministic logic = zero tokens)
            └── references/         (templates loaded on demand)

RULES PER ARTIFACT:

1) CLAUDE.md — a CONCISE operating manual (every line is reloaded each turn):
   - Stack + REAL detected commands (nothing invented).
   - Code conventions + commit style.
   - Sensitive zones "DO NOT MODIFY without human validation".
   - "Memory & discipline" section: read CLAUDE.md + session_state.md at the start
     of each phase, write decisions at the end, run /clear between phases.
   - Working rules: PLAN before implementing, strict scope, STOP +
     "BLOCKED: [reason]" after 2 failures, model choice per task type.
   - No inventing commands or paths that were not verified.

2) Sub-agents (.claude/agents/*.md) — required YAML frontmatter:
   - `name`, `description` (says WHEN to use it), `model`, `maxTurns`, `tools`.
   - Model choice: the most capable for architecture (Plan); a fast/cheap model
     to explore/review/test.
   - `tools` RESTRICTED to the strict minimum (read agents = read-only).
   - Body: 100–300 words max, with an explicit STOP CONDITION.
   - Use real model names available in this environment; if unsure, use generic
     aliases (opus/sonnet/haiku) and document it.

3) settings.json — `env` + `permissions`:
   - `env.DISABLE_NON_ESSENTIAL_MODEL_CALLS = "1"`.
   - `permissions.allow`: REAL test/lint/typecheck commands + git status/diff
     + execution of skill scripts.
   - `permissions.deny`: reading .env/secrets, destructive commands (rm -rf),
     git push.

4) commands (.claude/commands/*.md) — reusable slash commands, short, read-only
   when analytical (e.g. review-pr delegates to code-reviewer).

5) skills (.claude/skills/*/SKILL.md) — ONLY if a real repetitive need exists in
   THIS project (e.g. scaffolding an endpoint/component/module):
   - Clear `description` about the trigger (progressive disclosure).
   - Repetitive logic in `scripts/` (deterministic, zero tokens).
   - Bulky details in `references/` (loaded on demand).

6) session_state.md — externalized memory: objective, progress (checklist),
   table of involved files, out-of-scope, decisions made, blockers, next action
   ready to paste.

7) .mcp.json — only if relevant:
   - SCOPED MCP servers (never the repo root).
   - Include a semantic-search server (vector pre-filtering) as a placeholder if
     the corpus is large, with an install comment.

CROSS-CUTTING PRINCIPLES TO RESPECT EVERYWHERE:
- Externalized memory (files) over conversational context.
- Semantic pre-filtering before reading large files.
- Session discipline: phases + context reset, state written in black and white.
- Token economy: restricted tools, adapted models, bounded maxTurns, explicit
  stop conditions.
- Create NO superfluous file: a minimal, right-sized harness beats an overloaded one.

FINAL DELIVERABLE:
- Actually create the files.
- End with a recap: generated tree + 1 line per file + items the team should
  customize (models, commands, domain skills).

Start now with PHASE 1.
```

---

## How to use it

- Run it from the **project root**. Prefer a capable model for Analyze/Plan
  (`/model opus` or `/model sonnet`), then switch down for generation.
- The prompt **forces a validation gate** between plan and generation — you stay
  in control and avoid useless skills.
- It **self-scales**: a minimal harness for a small script, a full harness for a
  monorepo.
- **Reusable anywhere**: no technology is hard-coded; everything is detected.
