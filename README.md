<p align="center">
  <img src=".github/assets/banner.png" alt="Sparse: the right model for every task" width="900" />
</p>

<p align="center">
  <img alt="Platforms" src="https://img.shields.io/badge/macOS%20%C2%B7%20Windows-0a0d0b?style=flat-square&logo=electron&logoColor=3BE27A" />
  <img alt="MCP server" src="https://img.shields.io/badge/MCP-sparse--mcp-0a0d0b?style=flat-square" />
</p>

<p align="center">
  <strong>An AI assistant and coding agent that picks the right model for every task.</strong>
</p>

<p align="center">
  Sparse sits between you and every AI model you have access to. Ask it a question and it answers;
  point it at a project and it plans the work, turns each change into a reviewable diff, checks it, and learns
  what works on your repo. Behind the scenes it routes each step to the model that fits it: fast, cheap
  models for routine work, frontier models for the hard parts. You stop choosing models. You just build.
</p>

<p align="center">
  <img src=".github/assets/demo-app.gif" alt="Sparse desktop app building and testing a tip calculator from one prompt" width="900" />
  <br />
  <sub>One prompt in the desktop app: planned, built, verified, and previewed. Real run, fast-forwarded.</sub>
</p>

<p align="center">
  <a href="#download"><strong>Download</strong></a> ·
  <a href="#quick-start">Quick start</a> ·
  <a href="#how-routing-works">How routing works</a> ·
  <a href="#evidence">Evidence</a> ·
  <a href="https://github.com/Sparse-Agent/Sparse/issues/new/choose">Report a bug</a>
</p>

---

## Why Sparse

- **One assistant, every model.** One OpenRouter key gives Sparse the whole catalog: Anthropic, OpenAI, Google, DeepSeek, Moonshot, Z.ai and more. Its own agent runs the work, so there's no other tool to install. Prefer local? Point it at Ollama.
- **Chat or code.** Questions, emails, summaries and research get one well-routed answer. Coding tasks get a full pipeline: plan, edit, validate, review. "Work on a copy" drafts edits in a git worktree; Ask first and Full access edit your files in place. Shell commands the agent runs are not isolated.
- **Spend where it matters.** Most of a coding session isn't hard. Renames, boilerplate and "where is X?" go to fast, cheap models; deep reasoning, refactors and architecture get the frontier ones.
- **See every decision.** Each model pick is logged with its reason, and routing improves from the outcomes of your own runs.
- **A full desktop app.** Chats, plans, loops, sessions, cost tracking with a monthly budget, a built-in image studio, browser and terminal in one place.

## How it works

1. **Answer or work.** Every prompt is sorted into a conversation (wants an answer) or a task (wants changes to your project). Conversations get one well-routed reply, with read-only and web tools only.
2. **Plan.** For real work, a planner splits the prompt into sub-tasks with dependencies and assigns each one a model, with quality floors for risky or architectural changes.
3. **Run and check.** Sub-tasks run in parallel with their own lean context. A step that fails its checks is retried on a stronger model, and a final pass reviews the merged result.
4. **Learn.** Routes that work on your repo get preferred; routes that keep failing get avoided.

Pin a **Lead** (planning and hard work) and a **Workhorse** (mechanical work), or let Sparse balance cost and quality for you.

| Mode | What it does | Edits files? | Output |
|------|--------------|:------------:|--------|
| `agent` | Does the work and hands you a reviewable diff | ✓ | Diff, check results, report |
| `plan` | Quick, phased plan; asks at most one round of questions | | Phased plan |
| `masterplan` | Research-grade plan with dual planners and a judge | | Cited plan |
| `debug` | Gathers evidence and tests hypotheses | | Root-cause writeup |
| `review` | Read-only critique of a diff or PR | | Structured review |

`plan` upgrades itself to `masterplan` when it detects high-risk or architectural language.

## How routing works

Every model pick goes through one pipeline, and the pick is journaled with its reason:

```text
candidates → hard filters → pins → quality floors → repo memory → scoring → selection
```

| Stage | What it does |
|-------|--------------|
| **Candidates** | Models on a provider you have set up. Plan mode keeps only approved planners. |
| **Hard filters** | Drops models that can't do the job: no tools, no image input, context too small, privacy constraints. Batch-only, free-tier and floating aliases are never auto-routed. |
| **Pins** | Applies an explicit route, your Lead or Workhorse, or the planner's pick for that sub-task. |
| **Quality floors** | Sets how close to the best available model a candidate must be. The band tightens with effort, and for retries after a failure. |
| **Repo memory** | Excludes routes that keep failing in this repo and gives a small preference to routes that work. |
| **Scoring** | Ranks what's left by expected cost per successful result, including the cost of a repair attempt, plus expected latency. |
| **Selection** | Takes the top pick, or the pin, with two fallbacks from the same ranking. |

Two design choices matter most:

- **Thin evidence counts against a model.** Each model is scored on the lower confidence bound of its benchmark evidence, per task category and effort level, so a model with little data sits below a measured one rather than getting the benefit of the doubt.
- **A failed attempt is a cost.** A cheap model that often needs a repair can lose to a pricier model that succeeds the first time.

`sparse route "<prompt>"` shows the chosen route without running anything, and `sparse logs` shows the decision timeline of any past run.

## Evidence

**Status: Sparse's router has not yet been measured end to end against a single-frontier-model baseline.** That comparison is [the next evaluation](#planned-router-vs-baseline). What exists today is a measured view of the economics the router depends on, on Sparse's own evaluation suite.

### Measured: fixed models in Sparse's agent loop

October 2026 run: 96 coding tasks in 8 categories (bug fixes, features, refactors, tests, investigation, terminal, web, docs), one trial per model and task, 288 graded trials in all. The models were forced, so the router made no choices here.

| Model | Tasks passed | Mean cost per task | Cost per pass | Mean time per task |
|-------|:------------:|:------------------:|:-------------:|:------------------:|
| `anthropic/claude-opus-5` | 94/96 (98%) | $0.435 | $0.444 | 83 s |
| `google/gemini-3.8-flash` | 75/96 (78%) | $0.115 | $0.147 | 156 s |
| `deepseek/deepseek-v4-flash` | 71/96 (74%) | $0.014 | $0.019 | 137 s |

How these were graded and recorded:

- **Pass or fail comes from deterministic hidden checks** that the agent never sees. There is no LLM judge. Visible tests are restored from the seed before grading, so editing them doesn't help.
- **Cost is what the provider billed**, failed attempts included. 2 of 4,069 calls came back without a billed cost and aren't counted.
- **Conditions were equal and recorded per call:** the same token budget for each trial, each model pinned to one upstream host (0 violations), and the requested effort level confirmed on all 288 trials.
- **Infrastructure failures such as provider 429s were rerun**, not scored as failures. There were 21 in the first 288 trials.

### Simulated: what escalation could save

The table below replays those same trials as if a cheap model went first and a stronger one took over only on failure. **This is a calculation from the trials above, not a run of Sparse's router.**

| Policy | Tasks passed | Mean cost per task | Cost vs Opus alone | Mean time per task |
|--------|:------------:|:------------------:|:------------------:|:------------------:|
| Opus only | 94/96 | $0.435 | baseline | 83 s |
| DeepSeek, then Opus | 96/96 | $0.181 | 2.4× lower | 164 s |
| DeepSeek, then Gemini, then Opus | 96/96 | $0.116 | 3.8× lower | 195 s |

The 95% interval on the DeepSeek-then-Opus cost reduction is 1.8× to 3.7× (bootstrap over tasks). Read these as an optimistic estimate:

- **Failure detection is perfect in this replay.** It uses the hidden checks to decide when to escalate. A real run only has the project's own tests and review, so it will sometimes miss a failure or escalate needlessly.
- **It leaves out routing overhead.** Planning, verification and integration passes cost money in a real run and aren't in these numbers.
- **The tasks skew easy.** 57 of the 96 were passed by all three models, and easy tasks flatter a cheap-first policy.
- **Escalation costs time.** The cheaper policies are about twice as slow as Opus alone, because a failed attempt has to finish first.

### Limits of this evidence

- One trial per model and task, so flakiness isn't measured.
- 5 task prompts were clarified after the run, when all three models failed the same rule the prompt hadn't stated, and those tasks were rerun. The table includes the reruns. On the original prompts the passes were Opus 89, Gemini 72 and DeepSeek 68 of 96.
- The source and the task bank are private (the tasks stay private so they stay novel to the models being tested), so a third party can't rerun this today.

Where the live router stands today:

- It draws on public benchmark evidence and live prices. Results from this suite don't feed routing yet.
- Its success probabilities use a prior rather than a fitted calibration. A fitted candidate exists but isn't installed.
- It estimates a fixed token count per attempt, so its predicted cost differences between models reflect price only.

### Planned: router vs baseline

Not run yet. The design this evaluation needs:

| | |
|---|---|
| **Arms** | Sparse auto-routing, one frontier model alone, one cheap model alone. All in the same agent loop. |
| **Tasks** | The same hidden-check suite, with saturated tasks replaced by harder ones. |
| **Trials** | At least 3 per arm and task, paired by task. |
| **Task success** | Hidden-check pass rate, per category and overall, with confidence intervals. |
| **Cost** | Total billed cost per task and per success, including planning, review, retries and failed attempts. |
| **Latency** | Wall-clock time per task, time to first edit. |

Results will be published here whichever way they come out.

## Download

> **Availability:** no installer is attached to this repository yet, and `sparse-cli` isn't on npm yet. The instructions below describe how each one installs once published.

### Desktop app

**[Download the latest release](https://github.com/Sparse-Agent/Sparse/releases/latest)**

| Platform | File |
|----------|------|
| macOS (Apple Silicon) | `Sparse-<version>-arm64.dmg` |
| Windows | `Sparse Setup <version>.exe` |

The app is self-contained (no separate Node or CLI install needed) and updates itself: when a new version is out, a card appears bottom-left with **Update / Later**.

> Builds aren't code-signed yet. On macOS, right-click the app, choose **Open** and confirm. On Windows, if SmartScreen appears, click **More info**, then **Run anyway**.

### CLI

Requires **Node 24+**.

```bash
npm install -g sparse-cli
sparse          # or the short alias: cr
```

First launch walks you through adding your OpenRouter key. `sparse app` launches the desktop app from the terminal.

### MCP

`sparse-mcp` is a stdio MCP server that exposes Sparse as tools (`plan`, `masterplan`, `agent_run`, `route`, `validate`, and more) to any MCP host:

```json
{ "mcpServers": { "sparse": { "command": "sparse-mcp" } } }
```

The agent can also connect to external MCP servers: `sparse mcp add | list | remove | test`.

## Quick start

```bash
sparse                            # interactive REPL
sparse run "rename getCwd"        # one-shot, non-interactive
sparse route "fix typo"           # show the chosen route (no run)
sparse lead <model>               # pin the Lead model
sparse workhorse <model>          # pin the Workhorse model
sparse logs                       # browse run journals and decision timelines
sparse loop "make CI green"       # generate + run a self-correcting loop
```

In the REPL, type `/` for commands and `@` to reference files.

## The desktop app

The app wraps the same router in a full UI and runs a local daemon, so background work keeps going after you close the window.

| Area | What's there |
|------|--------------|
| **Chat** | Chat / Code switch, five modes, a card for every agent command, inline diffs with Accept / Undo, file attachments, voice input |
| **Overview** | Runs, spend, tokens, which models were used and why, activity heatmap |
| **Cost & Usage** | Spend by month, model and mode; budget pace; breakdowns by provider and task type |
| **Plans** | Drafts from Plan / Masterplan; refine in chat or hand to Agent |
| **Images** | Generate and edit with any OpenRouter image model, then iterate on-device: background removal, crop and upscale, trace to SVG, icon-set export |
| **Sessions** | The run journal: every decision a run made, as a timeline |
| **Loops** | Describe an outcome; Sparse generates a verifiable loop and runs it until the check passes |
| **Plugins** | Plugins, rules, skills and subagents, plus a marketplace |
| **Browser · Terminal · Changes** | An in-app browser, a real shell with live agent tabs, and the working-tree diff |
| **Settings** | Connections, model pins, run mode, monthly spending limit (default **$50/mo**, enforced by the daemon), data locations |

## Safety

Where file edits land depends on the access level you choose:

| Access | File edits |
|--------|------------|
| **Work on a copy** | Drafted in a git worktree; your checkout stays untouched until you accept them |
| **Ask first** | In place, with approval (the default for new installs) |
| **Full access** | In place, without asking |

**None of these is an OS sandbox.** Shell commands the agent runs execute as your user, with full filesystem and network access. The Ask first command allowlist is a best-effort guardrail, not a security boundary.

Repo instruction files (`AGENTS.md`, `CLAUDE.md`, `.cursorrules`, and project `.sparse` rules, skills and subagents) load only after you trust that folder.

## Configuration

Keys and preferences live in `~/.sparse/` and are editable from the app's Settings. An optional `sparse.config.{ts,js,json}` at the repo root adds per-repo overrides, such as the checks to run and spending ceilings:

```ts
export default {
  validators: { test: 'pnpm test', lint: 'pnpm lint', typecheck: 'pnpm typecheck' },
  costCeilings: { perRun: 2.50, perDay: 25.00 },
};
```

Optional Tavily or Brave keys unlock the agent's `web_search` tool; `web_fetch` works without them.

## Bugs and feature requests

[Open an issue](https://github.com/Sparse-Agent/Sparse/issues/new/choose). Include your OS, the app or CLI version (Settings, then About, or `sparse --version`), and steps to reproduce.

## About this repository

This is the public home for Sparse downloads and issue tracking. Sparse was previously called CodeRouter, and older `Code-Router/CodeRouter` links redirect here.

## License

Sparse is proprietary software, free to download and use. This repository hosts releases and issue tracking only; the source code is not public. See [LICENSE](LICENSE).
