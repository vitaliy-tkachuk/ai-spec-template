<div align="center">
    <h1>🪶 ai-spec-template</h1>
    <h3><em>Plan, scaffold, ship — with AI agents. Any stack.</em></h3>
</div>

<p align="center">
    <strong>A repo template that gives AI coding agents a simple, repeatable way to plan and implement changes — and to remember what they learned.</strong>
</p>

<p align="center">
    <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"/></a>
</p>

---

## 🤔 What is this?

AI agents are good at writing code and bad at remembering *why* the code looks the way it does. Every new session starts from zero, so they re-discover the same architecture, re-make the same decisions, and sometimes contradict what was decided last week.

This template fixes that with three things:

1. **A rulebook the agent reads first** — [`AGENTS.md`](AGENTS.md) tells any agent (Claude Code, Cursor, Codex, Copilot, Aider, …) how work is classified, what to read before coding, and where to write down what it learned.
2. **Three slash commands** — `/describe-project` (fill in the docs once), `/analyze` (plan a change), `/implement` (build it, verify it, document it, commit it).
3. **A tiny set of durable docs** — `docs/architecture.md` for cross-cutting decisions and one `docs/domains/<area>.md` per feature area. Nothing else accumulates: task files are local scratch that gets deleted when the work is done.

It is **stack-agnostic**. The template ships no code, no dependencies, and no build config — only Markdown and agent skills. The same workflow drives a web app, a CLI, a desktop app, a library, or firmware.

## 🚀 Step by step: from template to working project

Below is the whole loop, using a small example — a CLI tool called **notes**. Every step shows what you type and what you get back.

### Step 1 — Create your repo from the template

On GitHub: **Use this template → Create a new repository**. Or locally:

```bash
git clone https://github.com/<you>/ai-spec-template notes
cd notes && rm -rf .git && git init
```

**Result:** a repo containing only docs and skills — `AGENTS.md`, `docs/`, `.claude/skills/`, and a placeholder `README.template.md`. No source code yet.

### Step 2 — Describe the project (once)

Open the repo in Claude Code and run:

```
/describe-project
```

The agent asks a few questions: project name, one-line description, purpose, project type (web / desktop / CLI / library / other), and the stack. Stack questions adapt to the type — a CLI is asked for language, runtime, and distribution; a web app for framework, styling, database, auth, deployment. Answer `TBD` for anything you have not decided.

**Result:**

```
Described project: notes (CLI tool)
Updated: README.md (rendered from README.template.md), docs/architecture.md, AGENTS.md, .gitignore
Stack: Language: Go, Runtime / framework: cobra, Distribution: Homebrew + GitHub releases
Next: review the diff (`git diff`), commit the docs changes.
```

- `README.md` — **this file is replaced** by your project's README, rendered from `README.template.md`.
- `docs/architecture.md` — purpose, stack, and source layout filled in.
- `AGENTS.md` — the "Project-specific guidance" section now names your project.
- `.gitignore` — stack-specific ignores appended (e.g. `node_modules/`, `target/`, `__pycache__/`).

Nothing is installed and no code is written. This step only fills docs.

### Step 3 — Plan the first piece of work

```
/analyze propose a stack and scaffold the initial project structure
```

The agent reads the docs, classifies the work (Level 0–3, explained [below](#-how-work-is-classified)), and for anything non-trivial writes a **local task file** with a goal, acceptance criteria, files to touch, and a step-by-step plan.

**Result:**

```
Created: docs/tasks/T001_scaffold_project_structure.md
Next: review it, then say "implement T001" — or "clarify T001 ..." to refine it first.
```

Open the task file and read the plan. Change your mind about anything? Refine it in place:

```
/analyze clarify T001 — use urfave/cli instead of cobra
```

There is no approval button. Running `/implement` is the go-ahead.

### Step 4 — Implement it

```
/implement T001
```

The agent follows the plan, runs whatever checks your toolchain has (lint, typecheck, tests, build), and then does the bookkeeping for you.

**Result:**

```
Implemented: T001 — scaffold project structure
Files changed: go.mod, cmd/notes/main.go, internal/store/store.go, README.md, docs/architecture.md
Verified: build ✓ tests ✓ lint ✓
Distilled into: docs/domains/cli.md, docs/domains/storage.md
Deleted task: docs/tasks/T001_scaffold_project_structure.md
Committed: a1b2c3d feat: scaffold cli and file-backed store
```

- Code exists and passes its checks.
- `docs/domains/cli.md` and `docs/domains/storage.md` now record how those areas work, what was decided, and any gotchas.
- `docs/architecture.md` has the real stack rows instead of `TBD`.
- Your project `README.md` gained **Getting Started / Running Locally / Building & Releasing** sections with the commands that actually ran.
- The task file is gone. The knowledge lives in the domain docs, not in a pile of task history.
- One commit, staged with only the files that belong to this work.

### Step 5 — Repeat for every change

From here on, every change is the same two commands:

```
/analyze add a --tag filter to the list command
/implement T002
```

Small fixes skip the task entirely — just say what is wrong:

```
/implement fix the typo in the --help text
```

The agent fixes it, commits it, and updates a domain doc only if the fix taught it something durable.

### Step 6 (optional) — Build the knowledge graph

Once real source exists, build a [graphify](https://github.com/safishamsi/graphify) knowledge graph so agents can *ask* where code lives instead of sweeping the tree. See [Knowledge graph](#-knowledge-graph-optional). Every later `/analyze` and `/implement` gets cheaper.

## 📂 What you end up with

After a few tasks, the `notes` repo looks like this:

```
notes/
├── .claude/skills/            # /describe-project, /analyze, /implement
├── AGENTS.md                  # rules every agent follows, with your project's guidance
├── README.md                  # your project's README (this file is gone)
├── README.template.md         # source /describe-project renders README.md from
├── docs/
│   ├── architecture.md        # stack, boundaries, cross-cutting decisions
│   ├── coding-conventions.md  # style rules, incl. your language's specifics
│   ├── patterns.md            # patterns reused across domains
│   ├── feature-workflow.md    # the Level 0–3 rules the agent applies
│   ├── domains/
│   │   ├── cli.md             # durable knowledge, one file per feature area
│   │   └── storage.md
│   └── tasks/                 # local scratch, gitignored, empty between tasks
├── cmd/ internal/ ...         # your code, scaffolded and grown by /implement
└── graphify-out/              # optional knowledge graph (see below)
```

Three things stay true no matter how big the project gets:

- **Decisions are recorded where they are read** — cross-cutting ones in `docs/architecture.md`, local ones in the domain doc. There is no decisions folder.
- **Docs describe the current state, not history.** When behaviour changes, the doc is edited, not appended to.
- **Tasks never reach git.** `docs/tasks/` is gitignored. Task IDs are reused and never cited in docs or commit messages.

## 🎚️ How work is classified

`/analyze` picks the lightest process that fits. Full definitions live in [`docs/feature-workflow.md`](docs/feature-workflow.md).

| Level | Typical work | Task file? | What the agent leaves behind |
|---|---|---|---|
| **0** | Typo, broken import, lint error | No | The fix, committed. No docs. |
| **1** | Small bug in an existing area | No | The fix, a short note in the domain doc, a commit. |
| **2** | Small feature or isolated bug, 1–3 files | Yes, one | Code, updated domain doc, commit. Task deleted. |
| **3** | New feature area, multiple layers, new dependency | Yes, one or more | Code, new/updated domain doc, decision in `architecture.md`, commit. |

**Hard stop:** changes to the stack, database, auth, payments, deployment, or anything that contradicts a recorded decision always pause and ask for your approval, whatever the level.

## 🔌 Using it with other AI tools

Slash commands are Claude Code skills in `.claude/skills/`. Other tools follow the same workflow by reading `AGENTS.md` — the process is tool-agnostic. To get the same `/describe-project`, `/analyze`, `/implement` commands elsewhere, copy the skill folders into that tool's skills directory. Cursor (2.4+) uses the same `SKILL.md` format, so copying `.claude/skills/*` to `.cursor/skills/` is a near drop-in.

## 🕸️ Knowledge graph (optional)

Agents burn most of their tokens *finding* code. [graphify](https://github.com/safishamsi/graphify) turns the repo into a queryable graph so they ask instead of sweep — `AGENTS.md` tells them to query it first and read in full only what the query surfaces. Without a graph the workflow simply falls back to plain file reads.

Install once per machine:

```bash
uv tool install graphifyy       # or: pipx install graphifyy / pip install graphifyy
graphify install                # copy the /graphify skill into your agent's config dir
```

Then once per repo, from inside it, as soon as there is real source to index:

```bash
/graphify .                     # in Claude Code; CLI equivalent: graphify extract .
                                # → builds graphify-out/ (graph.json, GRAPH_REPORT.md, graph.html)
graphify hook install           # post-commit/post-checkout auto-rebuild + graph.json merge driver
git add graphify-out && git commit -m "chore: add knowledge graph"
```

Day to day:

- 🔎 `graphify query "<question>"` — cross-file answer; also `path`, `explain`, `affected`.
- ♻️ The code graph rebuilds itself after every commit (AST only, no LLM, no tokens). You **commit** that rebuild only on a cadence: when files or modules were added, removed, renamed, or moved, when a new domain landed, after doc-heavy work (run `graphify . --update` first), or roughly weekly. Not for fixes inside existing files. `graphify-out/` sitting modified between refreshes is expected; commit it on its own (`chore: refresh knowledge graph`), and `git checkout -- graphify-out/` if a dirty graph blocks a branch switch.
- 🧷 `graphify hook install` writes to `.git/hooks/` and local git config, so **every clone needs it re-run** (and CI, if CI should keep the graph fresh) — it is not carried by the commit. The AST layer needs no API key; only the semantic layer and community labels do (e.g. `GEMINI_API_KEY`). `.gitattributes` registers a union merge driver so `graph.json` does not conflict on every merge.

<details>
<summary><strong>What to commit from <code>graphify-out/</code></strong></summary>

The rule: *commit what costs tokens to recreate, ignore what a CPU regenerates for free.*

| Commit | Ignore (already in `.gitignore`) |
|---|---|
| `graph.json` — what agents query | `cache/ast/` — local AST extraction, purged on every version bump |
| `GRAPH_REPORT.md` — human-readable summary | `cache/stat-index.json` — machine-local stat signatures |
| `.graphify_labels.json` — LLM-named communities | `cost.json` — local token accounting |
| `manifest.json` — content hashes | `.graphify_python`, `.graphify_root` — absolute paths of whoever built it |
| `cache/semantic/` — LLM-derived per-file extractions | |

`manifest.json` and `cache/semantic/` matter more than they look: a fresh clone has new mtimes for every file, so without them `graphify extract` treats the whole corpus as new and **re-bills semantic extraction**. `graph.html` is regenerable — commit it if you want the viz browsable from a clone, ignore it if you would rather not carry the churn.

</details>

## 📚 Key docs

- 🤖 [`AGENTS.md`](AGENTS.md) — canonical instructions for AI agents
- 🪜 [`docs/feature-workflow.md`](docs/feature-workflow.md) — complexity levels (0–3), task lifecycle, commit rules
- 🏛️ [`docs/architecture.md`](docs/architecture.md) — system architecture and cross-cutting decisions
- ✍️ [`docs/coding-conventions.md`](docs/coding-conventions.md) — coding conventions
- 🧩 [`docs/patterns.md`](docs/patterns.md) — cross-domain reusable patterns
- 🧭 [`docs/domains/`](docs/domains/) — per-domain durable knowledge (the permanent record)

## 📄 License

[MIT](LICENSE)
