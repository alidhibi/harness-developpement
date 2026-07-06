# Claude Code — Best Practices Guide

**Cost control, memory architecture, Skills & AI-guided development harness**

*VS Code edition (Claude Code extension) — version 1.2*

---

**Author: Ali DHIBI**
Fractional CTO & Strategic Advisor — AI & LLM Expert (Granite, LangGraph) — Cloud Architecture & Cybersecurity
Site: https://alidhibi.me

---

## Table of Contents

1. CLI vs VS Code extension
2. The `/usage` issue and usage tracking
3. Architecture principles: externalized memory, semantic pre-filtering, session discipline
4. Controlling context size
5. Limiting sub-agent usage
6. Precise prompts = short sessions
7. Stopping self-correction loops
8. MCP tooling
9. Choosing the right model per task
10. Monitoring and alerts
11. Skills (Agent Skills)
12. Commands vs Sub-agents vs Skills
13. The AI-guided development harness
14. The Explore → Plan → Implement → Verify cycle
15. Ready-to-commit template files
16. Setup checklist
17. Operational summary
18. About the author

---

## 1. CLI vs VS Code extension

Everything living in the `.claude/` folder (agents, settings, MCP, commands, skills) is **identical** between the CLI and the VS Code extension. The only differences concern **launch flags** (`claude --model`, `--max-turns`, etc.), which **do not exist** in the extension UI: everything goes through **slash commands** and **config files**.

| Mechanism                 | CLI (`claude ...`)          | VS Code extension               |
| ------------------------- | --------------------------- | ------------------------------- |
| Reduce context            | `/compact`                  | `/compact` in chat              |
| Start fresh               | `/clear`                    | `/clear` in chat                |
| See cost                  | `/cost`                     | `/cost` in chat                 |
| Change model              | `/model` or `--model`       | `/model` in chat                |
| Sub-agents                | `.claude/agents/*.md`       | Identical (versioned files)     |
| Skills                    | `.claude/skills/*/SKILL.md` | Identical (versioned files)     |
| Limit turns               | `--max-turns` or `maxTurns` | `maxTurns` in config only       |
| Environment variables     | shell export                | `settings.json` or terminal     |
| Usage tracking            | `ccusage`, `/cost`          | Identical                       |
| Spend limits              | Anthropic Console           | Identical                       |

**Golden rule:** invest in the `.claude/` files — they are shared by the whole team and work everywhere.

---

## 2. The `/usage` issue and usage tracking

The `/usage` command shows **subscription quotas** (the 5-hour windows of Pro / Max plans). It only works if you are **authenticated with a Claude subscription account**.

The message `Usage tracking is only available for Claude AI subscribers.` means you are authenticated **via an API key (Console / pay-per-use billing)**. This is normal: on the API there is no plan quota to display.

| Situation                       | Recommended tracking tool                                    |
| ------------------------------- | ------------------------------------------------------------ |
| API / Console billing           | `/cost` + `ccusage` + Console dashboard (the team standard)  |
| Individual Pro / Max plan       | `/usage` (after `/login` with the subscription account)      |

**Check your authentication method:**

```bash
# In the Claude Code chat
/status        # shows the active auth method (API key vs subscription)
/login         # restarts the auth flow

# In the VS Code integrated terminal
echo $ANTHROPIC_API_KEY        # if set → API mode (no /usage)
```

> For a team billed per usage, stay on the API and adopt **`ccusage` + Console** as the tracking standard. Console tracking is superior to `/usage` for budget steering anyway.

---

## 3. Architecture principles: externalized memory, semantic pre-filtering, session discipline

These three principles form the foundation of an economical and reliable use of Claude on long missions. They move continuity **out** of the conversation window.

### 3.1 Externalized memory, not conversational

The model does not "remember" an ever-growing conversation. Instead rely on:

- A **permanent instruction file** (`CLAUDE.md`) acting as the **operating manual**.
- A set of **structured memory files** (`session_state.md`, decision notes, ADRs) that **persist across sessions**.

The cycle becomes: **read a documented state → work → write conclusions to memory → start fresh with a clean context**. You gain both in **cost** (no bloating history) and in **reliability** (state is explicit, versioned, reviewed).

| Approach                              | Cost           | Reliability on long sessions        |
| ------------------------------------- | -------------- | ----------------------------------- |
| Ever-growing conversational context   | Grows endlessly| Degrades (drift, forgetting)        |
| Externalized memory + reset           | Stable and low | High (state written in black & white)|

**Example prompt:**

```text
"Before starting, read CLAUDE.md and session_state.md. At the end of the task,
write your conclusions and decisions into session_state.md (modified files,
architecture choices, blockers), then we will /clear."
```

### 3.2 Semantic pre-filtering before any read

Rather than loading an entire file or document into context, **index the corpus upfront in a vector store**, and have the agent **query that store** to retrieve only the **truly relevant snippets**.

- Concrete result: often **10× fewer** tokens loaded uselessly.
- You lose nothing in relevance — the opposite: you **target better**.

**Concrete implementation in Claude Code / VS Code:**

1. Index the corpus (codebase, docs, tickets, wiki) as **embeddings** in a vector store (pgvector, Qdrant, LanceDB, etc.).
2. Expose a **semantic search via an MCP server** (see section 8 and the `.mcp.json` example).
3. Discipline the agent: **query the store first**, only read full files as a last resort.

| Without pre-filtering                | With semantic pre-filtering          |
| ------------------------------------ | ------------------------------------ |
| Read whole files/docs                | Retrieve only the useful snippets    |
| Heavy, costly, noisy context         | Light, targeted, precise context     |
| Risk of missing info in the noise    | Better relevance                     |

**Example prompt:**

```text
"Before reading anything: query the semantic search server (mcp semantic-search)
with the query 'refund validation'. Load only the returned snippets. Only read a
full file if a snippet is insufficient, and say so explicitly."
```

### 3.3 Session discipline

Split the mission into **sequential phases**, each in **its own session**, with a **context reset** (`/clear`) between phases. Continuity no longer relies on "what the model has in mind" but on what is **written in black and white** in externalized memory.

This is what avoids the **silent context accumulation** that hurts both cost and precision on long sessions.

**Per-phase discipline loop:**

```text
1. /clear                      (clean context)
2. "Read CLAUDE.md + session_state.md"   (reload state)
3. Phase work                  (strict scope)
4. "Update session_state.md"             (persist state)
5. /clear → next phase
```

> **Synthesis of the 3 principles:** memory lives in **files** (not the chat), you **filter semantically** before reading, and you **reset context** between phases. Controlled cost, increased reliability.

---

## 4. Controlling context size

Every token sent **and** received is billed, and context grows each turn. This is the main cost lever.

- Avoid unnecessary large files (verbose `CLAUDE.md`, massive imports, lockfiles).
- Use `/compact` regularly to summarize the running history.
- Use `/clear` when switching tasks entirely (cheaper than `/compact`).
- Prefer **short, focused sessions** over one long catch-all session.

**VS Code specifics:**

- Close unnecessary tabs: depending on your config, editor context may be added automatically.
- Use explicit `@file` references instead of letting Claude read a whole folder.
- Watch the context window fill indicator in the chat panel.

**Example prompts:**

```text
# Compact while targeting what to keep
"/compact keeping only the architecture decisions and the list of modified files.
Drop the intermediate test logs."

# Target context instead of loading everything
"Read only @src/auth/login.ts and @src/auth/session.ts. Don't explore the rest
of the auth folder for now."
```

---

## 5. Limiting sub-agent usage

Each sub-agent is spawned via the **Task tool**: Claude creates a fresh instance with its own context, restricted tools, and model. When it finishes, **only a summary** flows back to the main session.

Cost impact is **linear**: 10 parallel agents burn the quota 10× faster. An agent A calling B calling C causes **exponential explosion**.

**Antipatterns to avoid:**

- **Sub-agent prompts too long**: aim for 100–300 words max. Beyond that, you aren't specializing, you're reloading the same context in each window.
- **Parallelizing dependent tasks**: if B needs A's output, run sequentially.
- **Letting Claude decide alone** on fan-out: explicitly define when to orchestrate.

**Team arbitration rule:**

| Task type              | Agent profile         | Model  |
| ---------------------- | --------------------- | ------ |
| Reading / exploration  | Explore (read-only)   | Haiku  |
| Architecture analysis  | Plan (no writing)     | Opus   |
| Implementation         | general (all tools)   | Sonnet |
| Tests / verification   | Verify (bounded)      | Haiku  |

**Example prompts:**

```text
# Explicitly delegate to a sub-agent
"Use the codebase-explorer agent to locate where Stripe webhooks are handled.
Modify nothing, just report the involved files."

# Force sequential on dependent tasks
"Step 1: architect agent to plan the migration. STOP, wait for my validation.
Step 2 only after validation: we implement."

# Forbid fan-out
"Handle these 3 files yourself, sequentially. Do not spawn sub-agents."
```

---

## 6. Precise prompts = short sessions

A vague prompt pushes Claude to explore and iterate, generating more tokens to converge.

- Clearly define the **scope**: involved files, what **not to touch**, expected format.
- Use a **checkpoint** (`session_state.md`) to resume without re-contextualizing.
- In VS Code, **select the relevant code** in the editor before asking.

**Example prompts:**

```text
# Vague prompt (avoid)
❌ "Improve error handling"

# Precise prompt (prefer)
✅ "In @src/api/users.ts only, replace `throw new Error(...)` with our AppError
   class (see @src/errors/AppError.ts). Don't touch function signatures or tests.
   Expected format: a diff."

# Resume from a checkpoint
"Read session_state.md and resume the current step. Strict scope to the listed
files. STOP and report if blocked after 2 attempts."
```

---

## 7. Stopping self-correction loops

If Claude fails and retries in a loop (failing tests, lint errors), tokens pile up silently.

**Guardrails, by reliability in the extension:**

| Guardrail                        | Scope            | Extension reliability |
| -------------------------------- | ---------------- | --------------------- |
| Stop condition in the prompt     | Universal        | Excellent             |
| `maxTurns` in agent config       | Sub-agents       | Good                  |
| Manual interruption (Stop)       | Current session  | Manual                |

> The CLI flags `--max-turns` and `--no-auto-fix` **do not exist** in the extension UI. Control goes through `maxTurns` (agent config) and, above all, an **explicit stop condition in every prompt**.

**Example prompts:**

```text
❌ "Fix the failing tests"

✅ "Fix the failing tests. If you can't fix them in 2 attempts, stop and output:
   BLOCKED: [reason]. Don't retry further."

✅ "Fix the lint error in @src/utils/date.ts. Single pass. If the error persists,
   STOP and explain why without retrying."
```

---

## 8. MCP tooling

MCP tools (filesystem, terminal, semantic search, etc.) generate `tool_result` entries that **stay in history** and grow fast.

- Limit reads of large files via MCP.
- Restrict allowed `tools` per agent.
- Always **scope** MCP servers (e.g. `./src`, never the repo root).
- Regularly audit which MCP servers are active.
- **Good MCP use:** expose a **semantic search** (vector store) for the pre-filtering of section 3.2, instead of loading whole documents.

**Example prompts:**

```text
"Before reading via the MCP filesystem server, first list matching files with
Glob and only load those under 200 lines."

"Query the MCP semantic-search server first; only read a full file if the
snippets are insufficient."
```

### 8.1 Recommended MCP servers

These MCP servers concretely implement the principles of section 3 (externalized
memory + semantic pre-filtering). They are **recommended, not mandatory**: always
verify the real config (command, `args`, environment variables) in the server's
README before committing, and respect your project's security constraints.

| MCP server             | Use                                                | Link |
| ---------------------- | -------------------------------------------------- | ---- |
| codebase-memory-mcp    | Codebase memory + semantic search (RAG)            | https://github.com/DeusData/codebase-memory-mcp |
| server-filesystem      | Scoped file access (targeted reads)                | @modelcontextprotocol/server-filesystem |

**Example `.mcp.json` placeholder** (fill in with the server's real docs):

```json
{
  "mcpServers": {
    "codebase-memory": {
      "//": "See https://github.com/DeusData/codebase-memory-mcp — verify command + env vars in the README before committing.",
      "command": "<see repo README>",
      "args": ["<see repo README>"],
      "env": {
        "//": "Fill in per docs (index, embeddings model, etc.)"
      }
    }
  }
}
```

> ⚠️ Never paste an invented MCP config. A misconfigured, unscoped server, or one
> pointing at the repo root, can instead blow up your context and costs.

---

## 9. Choosing the right model per task

The recommended strategy relies on three tiers: **Opus** for complex reasoning, **Sonnet** for the bulk of the work, **Haiku** for simple tasks.

**Manual switch in session** (slash commands, works in the extension):

```text
/model haiku     # verification, lint, simple reading
/model sonnet    # standard task (~80% of cases)
/model opus      # architecture, deep debugging, critical decisions
```

The `opusplan` mode (Opus to plan, switches to Sonnet to code) is selected via `/model`.

**Model / task matrix to share:**

| Task                                 | Recommended model |
| ------------------------------------ | ----------------- |
| Codebase exploration, reading        | Haiku             |
| Standard refactor, adding a feature  | Sonnet            |
| PR review, test generation           | Sonnet            |
| Architecture, complex debugging      | Opus (plan phase) |
| Infra migration, critical decisions  | Opus              |

**Example prompts:**

```text
# Simple task in Haiku
"/model haiku"
"List all TODO and FIXME under @src and group them by file."

# Reasoning in Opus, then switch down to code
"/model opus"
"Design the migration strategy for the users table. Give the plan, code nothing."
# ... after validation ...
"/model sonnet"
"Implement step 1 of the validated plan."
```

---

## 10. Monitoring and alerts

**ccusage — see where tokens go** (run in the integrated terminal):

```bash
npx ccusage@latest daily --breakdown    # per-day usage, broken down by model
npx ccusage@latest blocks --live        # 5h billing windows in real time
npx ccusage@latest monthly              # monthly view
npx ccusage@latest daily --since 20260501 --until 20260601   # analyze a spike
```

**`/cost` in session**: cost and tokens consumed in the current session.

**Anthropic Console**: `console.anthropic.com` → **Usage** (charts) and **Settings > Limits** (monthly cap + email alerts at 50%, 80%, 100%).

**Background-noise reduction**: set `DISABLE_NON_ESSENTIAL_MODEL_CALLS=1` in `settings.json`.

**Indicators to track per sprint:**

| Indicator                                 | Source                      |
| ----------------------------------------- | --------------------------- |
| Tokens / session per dev                  | `ccusage daily --breakdown` |
| Unjustified Opus sessions                 | Logs / Console              |
| Sub-agent fan-out (20+)                   | Session observation         |
| Compaction cascades (long sessions)       | Session observation         |
| Retry loops on resubmitted context        | Session observation         |

Designate a **budget owner** in charge of weekly monitoring.

---

## 11. Skills (Agent Skills)

A **Skill** is a folder packaging reusable expertise: instructions (`SKILL.md`), and optionally **scripts** and **resources** (templates, schemas). Claude automatically loads a skill when its `description` matches the current task.

**Why Skills are a cost lever (progressive disclosure):**

- Only the **metadata** (`name` + `description`) stays permanently in context — very light.
- The **`SKILL.md` body** loads only when the skill is actually triggered.
- **Scripts** run **deterministically**: no tokens generated for repetitive logic.

**Skill structure:**

```text
.claude/skills/
└── skill-name/
    ├── SKILL.md          # required: frontmatter + instructions
    ├── scripts/          # optional: executable scripts (deterministic)
    │   └── generate.mjs
    └── references/       # optional: templates, schemas, examples
        └── template.ts
```

**Available scopes:**

| Scope     | Location            | Use                              |
| --------- | ------------------- | -------------------------------- |
| Project   | `.claude/skills/`   | Versioned, shared by the team    |
| Personal  | `~/.claude/skills/` | Your personal skills, unversioned|

**Best practices:**

- The `description` must say **when** to use the skill (it triggers loading).
- Keep `SKILL.md` concise; offload bulky details to `references/`.
- Put repetitive, deterministic logic in `scripts/`.

**Trigger a skill:**

```text
/skills                       # lists loaded skills (version dependent)
"Use the create-endpoint skill to add a GET /invoices route."
```

---

## 12. Commands vs Sub-agents vs Skills

| Criterion             | Slash Command             | Sub-agent                 | Skill                                 |
| --------------------- | ------------------------- | ------------------------- | ------------------------------------- |
| Trigger               | Explicit (`/command`)     | Delegation (Task tool)    | Auto (description) or explicit        |
| Context               | Current session           | New isolated window       | Current session (disclosure)          |
| Presence cost         | Zero until used           | Creates a full context    | Metadata only at rest                 |
| Ideal for             | Human-triggered workflow  | Heavy task to isolate     | Reusable expertise, scripts           |
| Can execute code      | Yes (via prompt)          | Yes (per tools)           | Yes (packaged scripts)                |

**Simple rule:**

- **Command** = "I want to run this on demand" (e.g. `/review-pr`).
- **Sub-agent** = "delegate this big task in a separate context".
- **Skill** = "apply this procedure/expertise whenever the topic appears".

---

## 13. The AI-guided development harness

The **harness** is the versioned framework that structures how the team works with Claude. It makes sessions reproducible, cheaper, and predictable.

**Harness components (all versioned in Git):**

| File / folder           | Role                                                          |
| ----------------------- | ------------------------------------------------------------- |
| `CLAUDE.md`             | Concise project memory (operating manual)                     |
| `.claude/agents/`       | Specialized sub-agents (model + maxTurns + restricted tools)  |
| `.claude/commands/`     | Reusable, proven slash commands                               |
| `.claude/skills/`       | Skills: expertise + deterministic scripts (disclosure)        |
| `.mcp.json`             | Scoped MCP servers (including semantic search)                |
| `.claude/settings.json` | Shared env vars and permissions                               |
| `session_state.md`      | Externalized memory / checkpoint for long tasks               |

**Why it reduces cost:** essential context is in `CLAUDE.md`, repetitive behaviors are in agents/commands/skills, memory is in external files, and the model is always task-appropriate. The upfront investment pays back within a few sprints.

**Team best practice:** treat the harness as code — reviewed in PRs, improved collectively, with a designated maintainer.

---

## 14. The Explore → Plan → Implement → Verify cycle

| Phase     | Agent / tool         | Model  | Writing | Guardrail                      |
| --------- | -------------------- | ------ | ------- | ------------------------------ |
| Explore   | `codebase-explorer`  | Haiku  | No      | Read-only + semantic pre-filter|
| Plan      | `architect`          | Opus   | No      | Read-only + human validation   |
| Implement | general session (+ skills) | Sonnet | Yes | Strict scope                   |
| Verify    | `test-runner`        | Haiku  | Yes     | `maxTurns` + STOP/BLOCKED      |

Reset context (`/clear`) between phases; state flows through `session_state.md`.

**Example prompts per phase:**

```text
# Explore
"Use the codebase-explorer agent. First query semantic-search on 'payment
validation', then report the key files (path:line). Modify nothing."

# Plan
"/model opus"
"Use the architect agent to plan adding a refund system. Read-only. End with
PLAN READY and wait for my validation."

# Implement (after validation, fresh context)
"/clear"
"Read session_state.md. Implement step 1 of the plan. Scope: @src/payments/refund.ts
+ its test. Use the create-endpoint skill."

# Verify
"Use the test-runner agent. 2 attempts max, then STOP + BLOCKED. Then update
session_state.md."
```

---

## 15. Ready-to-commit template files

Target tree:

```text
.
├── CLAUDE.md
├── session_state.md
├── .mcp.json
└── .claude/
    ├── settings.json
    ├── agents/
    │   ├── architect.md
    │   ├── codebase-explorer.md
    │   ├── code-reviewer.md
    │   └── test-runner.md
    ├── commands/
    │   └── review-pr.md
    └── skills/
        └── create-endpoint/
            ├── SKILL.md
            ├── scripts/
            │   └── scaffold.mjs
            └── references/
                └── endpoint.template.ts
```

### CLAUDE.md

```text
# Project — Claude Code Memory (operating manual)
# Keep this file CONCISE. Every line is reloaded each session = cost.

## Stack
- Language / framework : <e.g. TypeScript, Node 20, React 18>
- Package manager      : <e.g. pnpm>
- Database             : <e.g. PostgreSQL via Prisma>

## Essential commands
- Install / Dev / Build / Test / Lint / Typecheck : pnpm <cmd>

## Code conventions
- ESLint + Prettier enforced (do not bypass).
- No `any` in TypeScript; type explicitly.
- Tests required for any new business function.
- Commits: Conventional Commits (feat:, fix:, chore:...).

## Memory & context (discipline)
- At the start of each phase: read CLAUDE.md + session_state.md.
- At the end: write decisions/state to session_state.md, then /clear.
- Pre-filter via the MCP semantic-search server before reading large files.

## Sensitive zones — DO NOT MODIFY without human validation
- src/auth/**, migrations/**, infra/**, .env*

## Working rules with Claude
- PLAN before implementing any non-trivial task.
- Strict scope to the mentioned files.
- If blocked after 2 attempts: STOP + `BLOCKED: [reason]`.
- Haiku to explore, Sonnet to code, Opus for architecture.
- Use the create-endpoint skill for any new API route.
```

### .claude/settings.json

```json
{
  "env": {
    "DISABLE_NON_ESSENTIAL_MODEL_CALLS": "1"
  },
  "permissions": {
    "allow": [
      "Read",
      "Grep",
      "Glob",
      "Bash(pnpm test:*)",
      "Bash(pnpm lint:*)",
      "Bash(pnpm typecheck:*)",
      "Bash(node .claude/skills/**)",
      "Bash(git status)",
      "Bash(git diff:*)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./**/secrets/**)",
      "Bash(rm -rf:*)",
      "Bash(git push:*)"
    ]
  }
}
```

### .claude/agents/architect.md

```yaml
---
name: architect
description: PLAN phase. Designs architecture and the implementation plan. Read-only, never codes.
model: claude-opus-4-1
maxTurns: 10
tools: Read, Grep, Glob
---
You are a senior software architect. Your role is to PLAN, never to implement.

Procedure:
1. Read session_state.md, then explore relevant code (targeted Read/Grep/Glob).
2. Identify constraints: dependencies, sensitive zones (see CLAUDE.md), risks.
3. Produce a structured PLAN:
   - Objective (1-2 sentences)
   - Files to create / modify (precise paths)
   - Files to NOT touch
   - Ordered, atomic steps
   - Test / verification strategy
   - Risks and required human decisions
4. ALWAYS end with: "PLAN READY — validate before implementation."

Constraints: READ-ONLY, no complete code, ask questions if ambiguous, stay
concise (one page max).
```

### .claude/agents/codebase-explorer.md

```yaml
---
name: codebase-explorer
description: Explores and maps the code to answer a question. Read-only, economical.
model: claude-haiku-4-5
maxTurns: 8
tools: Read, Grep, Glob
---
You answer an architecture question at minimal cost.

Procedure:
1. Use semantic search / Grep / Glob to TARGET before reading.
2. Read only relevant portions of files.
3. Synthesis: direct answer, key files (path:line), flow if useful.

Constraints: READ-ONLY, avoid whole large files, be brief.
```

### .claude/agents/code-reviewer.md

```yaml
---
name: code-reviewer
description: Focused code review on a file, diff, or PR. Read-only.
model: claude-haiku-4-5
maxTurns: 6
tools: Read, Grep, Glob
---
You are a senior code reviewer.

1. Read ONLY the provided files or diff.
2. Prioritized list: Blocking / Important / Minor.
3. Each point: file:line + short explanation + suggestion.

Constraints: no modifications, no commands, stop as soon as analysis is done.
If nothing to report: "OK — code compliant".
```

### .claude/agents/test-runner.md

```yaml
---
name: test-runner
description: Runs tests and reports failures. Fixes at most 2 times.
model: claude-haiku-4-5
maxTurns: 5
tools: Bash, Read, Edit
---
1. Run `pnpm test`.
2. If OK: "All tests pass" and STOP.
3. If failing: apply a MINIMAL, targeted fix, then rerun.
4. After 2 failed attempts: STOP + "BLOCKED: [summary + files]".
   DO NOT CONTINUE beyond 2 attempts.

Constraints: only strictly necessary code, never fake tests, stay within the
scope of the failure.
```

### .claude/commands/review-pr.md

```text
---
description: Runs a full code review on current changes
---
1. Run `git diff --staged` (or `git diff` if nothing is staged).
2. Delegate analysis to the code-reviewer agent.
3. Result grouped by severity (Blocking / Important / Minor).
4. Verdict: APPROVED / NEEDS CHANGES / BLOCKING.
Read-only, no modifications.
```

### .claude/skills/create-endpoint/SKILL.md

```yaml
---
name: create-endpoint
description: >
  Creates a new REST API endpoint following the project's conventions (routing,
  validation, error handling, test). Use it whenever the user asks to add a
  route, endpoint, or API handler.
---
# Skill: API endpoint creation

## When to use it
Any request to add an HTTP route (GET/POST/PUT/DELETE) on the API side.

## Procedure
1. Infer: HTTP method, path, input schema, output schema.
2. Generate the skeleton via the deterministic script (no manual generation):
   `node .claude/skills/create-endpoint/scripts/scaffold.mjs <method> <path>`
3. Adapt the generated file to the requested business logic.
4. Respect: Zod input validation, error handling via AppError, a minimal test
   (happy path + error case).
5. Never modify src/auth/** without human validation.

## Expected output
- 1 handler + 1 test + route registration + a summary of files.

## Details
See references/endpoint.template.ts for the reference skeleton.
```

### .claude/skills/create-endpoint/scripts/scaffold.mjs

```javascript
#!/usr/bin/env node
// Generates an endpoint skeleton deterministically (zero LLM tokens).
// Usage: node scaffold.mjs <method> <routePath>
// Example: node scaffold.mjs GET /invoices

import { writeFileSync, mkdirSync, existsSync } from "node:fs";
import { dirname, join } from "node:path";

const [, , methodArg, routeArg] = process.argv;
if (!methodArg || !routeArg) {
  console.error("Usage: scaffold.mjs <method> <routePath>");
  process.exit(1);
}

const method = methodArg.toUpperCase();
const name = routeArg.replace(/^\//, "").replace(/[\/:]/g, "-") || "root";
const handlerPath = join("src", "api", `${name}.ts`);
const testPath = join("tests", "api", `${name}.test.ts`);

const handler = `import { z } from "zod";
import { AppError } from "../errors/AppError";

export const ${name}Schema = z.object({
  // TODO: define the input schema
});

export async function ${name}Handler(req: unknown) {
  const input = ${name}Schema.parse(req);
  // TODO: business logic
  return { ok: true };
}
`;

const test = `import { describe, it, expect } from "vitest";
import { ${name}Handler } from "../../src/api/${name}";

describe("${method} ${routeArg}", () => {
  it("happy path", async () => {
    expect(true).toBe(true);
  });
  it("error case", async () => {
    expect(true).toBe(true);
  });
});
`;

for (const [p, content] of [[handlerPath, handler], [testPath, test]]) {
  if (existsSync(p)) {
    console.error(`SKIP (already exists): ${p}`);
    continue;
  }
  mkdirSync(dirname(p), { recursive: true });
  writeFileSync(p, content);
  console.log(`CREATED: ${p}`);
}

console.log(`\nTODO: register the ${method} ${routeArg} route in the router.`);
```

### .claude/skills/create-endpoint/references/endpoint.template.ts

```typescript
// Reference skeleton of an endpoint compliant with the project conventions.
// Loaded on demand by the skill (progressive disclosure).
import { z } from "zod";
import { AppError } from "../../src/errors/AppError";

export const InputSchema = z.object({
  // validated input fields
});

export type Input = z.infer<typeof InputSchema>;

export async function handler(rawInput: unknown) {
  const input = InputSchema.parse(rawInput); // 400 if invalid
  if (!input) {
    throw new AppError("INVALID_INPUT", "Invalid input", 400);
  }
  // business logic...
  return { ok: true };
}
```

### session_state.md

```text
# Session State — <task name>
# Externalized memory / resume checkpoint.
# Last update: <date> — Author: <dev>

## Overall objective
<2-3 sentences>

## Progress
- [x] Done step
- [ ] Current step: <precise description>
- [ ] Next step

## Involved files
| File                 | Status   | Note                       |
| -------------------- | -------- | -------------------------- |
| src/auth/login.ts    | modified | logic OK, tests to write   |
| src/auth/session.ts  | to create|                            |
| tests/auth.test.ts   | ongoing  | 2 cases left               |

## Out of scope (do not touch)
- migrations/**
- src/payments/**

## Decisions made
- <e.g. JWT in httpOnly cookie (security).>
- <e.g. Refresh token in DB, `sessions` table.>

## Blockers / pending
- BLOCKED: <description> — human decision required?

## Next action to run
"/clear then: read session_state.md and implement the current step, scope limited
to the listed files. STOP if blocked after 2 attempts."
```

### .mcp.json (with semantic search)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src"]
    },
    "semantic-search": {
      "command": "npx",
      "args": ["-y", "<your-vector-search-mcp-server>"],
      "env": {
        "VECTOR_DB_URL": "<your-vector-db-url>",
        "EMBEDDINGS_MODEL": "<embeddings-model>"
      }
    }
  }
}
```

---

## 16. Setup checklist

- [ ] Create the `.claude/` tree (agents, commands, skills) and commit.
- [ ] Adapt `CLAUDE.md` to the real stack (commands, sensitive zones, memory discipline).
- [ ] Adapt the `Bash(...)` commands in `settings.json` to the package manager.
- [ ] Adapt the `create-endpoint` skill (script + template) to the real framework.
- [ ] Index the corpus in a vector store + wire the `semantic-search` MCP server.
- [ ] Establish session discipline: `/clear` + re-reading `session_state.md` between phases.
- [ ] Verify auth (`/status`); tracking standard: `ccusage` + Console.
- [ ] Set a spend limit in the Console + alerts at 50 / 80 / 100%.
- [ ] Designate a budget owner (weekly `ccusage daily --breakdown` review).
- [ ] Generate this guide's PDF and share it at kickoff.

---

## 17. Operational summary

| Lever              | Key action                                                                        |
| ------------------ | --------------------------------------------------------------------------------- |
| Memory             | Externalized in files (`CLAUDE.md`, `session_state.md`), not the chat             |
| Pre-filtering      | Vector store + MCP semantic-search: load only useful snippets                     |
| Session discipline | Sequential phases + `/clear` between each, state written in black and white       |
| Context            | Regular `/compact`, concise files, targeted `@file` references                    |
| Sub-agents         | YAML configs with `model` + `maxTurns` + restricted `tools`                       |
| Skills             | Reusable procedures + deterministic scripts (progressive disclosure)              |
| Loops              | Explicit stop condition (`BLOCKED`) in every prompt                               |
| Model              | Haiku/Sonnet by default, Opus on conscious decision                              |
| Budget             | `ccusage` + `/cost` + Console spend limits + budget owner                        |
| Cycle              | Explore (Haiku) → Plan (Opus, validated) → Implement (Sonnet) → Verify (Haiku, bounded) |

---

## 18. About the author

**Ali DHIBI**
Fractional CTO & Strategic Advisor | AI & LLM Expert (Granite, LangGraph) | Cloud Architecture & Cybersecurity

Personal site: https://alidhibi.me

*This document may be shared within your team. For any questions on AI architecture, setting up an AI-guided development harness, or LLM strategy, contact the author via his site.*

*End of guide.*
