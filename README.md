<div align="center">

# 📐 Workflow Agent Skill Specification

> 💎 **Don't just save a prompt — design the task as a workflow that runs: resumable, testable, auditable.**

[![Framework](https://img.shields.io/badge/Framework-16_Modules-FF6B35?style=for-the-badge)](docs/specification.md)
[![Docs](https://img.shields.io/badge/Docs-22-2188FF?style=for-the-badge)](docs/)
[![Evals](https://img.shields.io/badge/Evals-5_Suites-brightgreen?style=for-the-badge)](evals/)
[![Examples](https://img.shields.io/badge/Examples-2_Skills-orange?style=for-the-badge)](examples/)
[![Agent Ready](https://img.shields.io/badge/Agent-Ready-412991?style=for-the-badge&logo=openai&logoColor=white)](CLAUDE.md)
[![License](https://img.shields.io/badge/License-MIT-2C3E50?style=for-the-badge)](LICENSE)
[![Stars](https://img.shields.io/github/stars/aiworkflowpro/workflow-agent-skill-spec?style=for-the-badge&logo=github&color=181717)](https://github.com/aiworkflowpro/workflow-agent-skill-spec/stargazers)

</div>

---

> 🎯 **Most Skills fail not because the model can't do the work, but because the task was never designed as a system that runs.**
>
> This repository is a specification, not a Skill. It defines how to build production-grade Workflow Agent Skills: set trigger boundaries → keep the entry short → break the work into steps → back judgment with scripts → persist state → test and audit before release. **16 modules, 22 docs, 5 eval suites, 2 worked examples.**
>
> Hand the spec to your Agent and let it create, review, or upgrade Skills for you — cross-platform, tied to no single client, and safe to distribute publicly.

<div align="center">

[🚀 Quick Start](#-quick-start) · [📚 Repository Map](#-repository-map) · [🤖 AI Workflow Scenarios](#-ai-workflow-scenarios) · [📖 Full Spec](docs/specification.md)

</div>

---

## 🤔 Why a Spec, Not a Prompt

> 💡 **A Skill is a repeatable workflow. If you often ask an Agent to research, review code, publish content, or organize files, that process deserves to be an engineered system — not a prompt you paste again and again.**

Many Skills fall short because the task was never designed to run. This specification breaks those failure modes apart, one module at a time:

| Common failure | What the spec fixes |
|---|---|
| 🎯 **Triggers too broad** | Defines when a Skill should — and should not — fire |
| 📄 **`SKILL.md` too long** | Keeps the entry short; layers the detail into `docs/` |
| 🌀 **Workflow describes wishes** | Requires inputs, outputs, completion criteria, and failure handling per step |
| 🧩 **Everything relies on prompts** | Separates deterministic work into scripts the model can call |
| 🔁 **One interruption = start over** | Standardizes `runs/`, `progress.json`, and a recovery flow |
| 🔓 **No pre-release check** | Ships trigger, function, recovery, security, and runtime eval suites |

> 💎 **The Agent handles judgment, scripts handle determinism, state handles recovery, tests handle release confidence.**

---

## 🚀 Quick Start

> 🎁 **You don't install this like a library — you hand it to your Agent.** Point Claude Code, Codex, or any Agent client at this repo and give it one of the tasks below.

| You want to… | Give your Agent | Reads |
|---|---|---|
| 🆕 **Build a new Skill** | "Build a `{task}` Skill against the Workflow Agent Skill Specification" | [specification.md](docs/specification.md) + [templates/](templates/) |
| 🔍 **Review a Skill** | "Check this Skill against the spec — is it ready for public release?" | [security.md](docs/security.md) + [quality-rubric.md](docs/quality-rubric.md) |
| ⬆️ **Upgrade a prompt** | "Turn this prompt into a reusable Skill per the spec" | [skill-file.md](docs/skill-file.md) + [workflow-steps.md](docs/workflow-steps.md) |
| ✅ **Validate locally** | `python3.12 scripts/validate-spec.py --root .` | [testing.md](docs/testing.md) |

> 💡 **Two entry points, one source of truth:** [`CLAUDE.md`](CLAUDE.md) and [`AGENTS.md`](AGENTS.md) give Agent clients their project context; [`README.md`](README.md) is for human readers. `AGENTS.md` points back to `CLAUDE.md` so there's only one set of rules to maintain.

---

## 📚 Repository Map

> 🎯 **A short entry point up top, the full framework in `docs/`, and machine-verifiable contracts to check against.**

```text
workflow-agent-skill-spec/
  README.md                 Public introduction (this page)
  CLAUDE.md / AGENTS.md     Agent project entry points
  docs/                     22 docs — the full specification
  templates/                Reusable SKILL.md / progress / runtime templates
  examples/                 A minimal Skill and a golden workflow Skill
  schemas/                  JSON Schemas for metadata, progress, runtime, triggers
  evals/                    Trigger, function, recovery, security, runtime cases
  scripts/validate-spec.py  Dependency-free local validator
```

Core documents:

| Document | Covers |
|---|---|
| 📖 [specification.md](docs/specification.md) | The full 16-module framework, quality tiers, and release gates |
| 📝 [skill-file.md](docs/skill-file.md) | `SKILL.md` entry, frontmatter, structure, lifecycle |
| 🔧 [workflow-steps.md](docs/workflow-steps.md) | Step naming, executors, inputs/outputs, verification |
| ⚙️ [configuration.md](docs/configuration.md) | L1/L2/L3 config layering and runtime declarations |
| 💾 [run-state.md](docs/run-state.md) | `runs/`, `progress.json`, and the recovery flow |
| 🧪 [testing.md](docs/testing.md) | Trigger, function, recovery, security, and regression testing |
| 🔒 [security.md](docs/security.md) | Public-release security boundaries |
| 🧭 [platform-adapters.md](docs/platform-adapters.md) | Adapting across Claude Code, Codex, Cursor, and more |

---

## 🤖 AI Workflow Scenarios

> 🚀 **The spec exists to be handed to an Agent.** Each scenario below is a real prompt you can paste into Claude Code or Codex — the Agent reads the spec and does the building, reviewing, or upgrading for you.

### 🔬 Scenario 1: Build a deep-research workflow

Turn a repeated research task into a resumable Skill:

```text
You (to Claude Code):

  Build a "deep research workflow" Skill against the Workflow Agent Skill Specification.
  It should: take a topic, break it into research questions, gather and organize sources,
  output a structured report, and record progress at each step so I can resume after an interruption.
  Generate SKILL.md, workflow/, config/runtime.json, evals/, and the example files per the spec.
```

🔬 **A one-off research prompt becomes a Skill that runs the same way every time — and survives an interruption.**

### 🧪 Scenario 2: Build a deep-analysis workflow

Package project, product, or competitor analysis:

```text
You (to Claude Code):

  Build a "deep analysis workflow" Skill against the spec.
  It analyzes a project, product, industry, or competitor: confirm the target first,
  gather background, output conclusions step by step, state evidence / risks / recommendations,
  and finally produce a reusable analysis report.
```

🧪 **Your analysis method stops living in your head and becomes an asset the whole toolchain can reuse.**

### 🔍 Scenario 3: Review a Skill before public release

Audit an existing Skill against the release gates:

```text
You (to Codex):

  Use the Workflow Agent Skill Specification to check whether this Skill is ready to publish.
  Path: {skill path}
  Focus on: when it triggers and when it shouldn't, whether the steps are clear,
  any hidden local paths / accounts / keys / private dependencies, whether it resumes after
  an interruption, whether it has test cases — then give a public-release verdict.
```

🔍 **Catch broad triggers, leaked paths, and untested steps before a stranger clones your repo.**

### ⬆️ Scenario 4: Upgrade a plain prompt into a Skill

Promote a prompt you keep reusing:

```text
You (to Claude Code):

  I only have a prompt and want to upgrade it into a reusable Skill. Use the spec to help.
  Prompt: {original prompt}
  Don't just save it — fill in: when to use the Skill, what input the user provides,
  how many steps it runs, what output it produces, how to verify the result, and how to resume.
```

⬆️ **A prompt is a wish; a Skill is a contract. This turns one into the other.**

### 👥 Scenario 5: Give a team one shared Skill format

Standardize how everyone builds workflows:

```text
You (to Codex):

  Validate every Skill in our /skills/ folder against the Workflow Agent Skill Specification.
  For each, run scripts/validate-spec.py, check it against schemas/skill.schema.json,
  and produce a table: pass/fail, missing sections, and the top fix for each Skill.
```

👥 **Instead of everyone writing their own prompts, the team's Agent workflows share one auditable format.**

### 🛠️ Who it's for

| Tier | You are… | Start with |
|---|---|---|
| 🟢 **Builder** | Someone turning a repeated task into a Skill | [specification.md](docs/specification.md) + [templates/](templates/) |
| 🟡 **Agent power user** | Handing this repo to an Agent to build for you | The scenarios above |
| 🟠 **Maintainer** | Shipping a public, safe, testable Skill repo | [security.md](docs/security.md) + [testing.md](docs/testing.md) |
| 🔴 **Team lead** | Giving a team one shared workflow format | [quality-rubric.md](docs/quality-rubric.md) + [schemas/](schemas/) |

> 💡 **Start by handing the spec to your Agent for one real task.** Adopt the schemas and eval suites once you're shipping Skills to other people.

---

## 🤝 Contributing

Open an [issue](https://github.com/aiworkflowpro/workflow-agent-skill-spec/issues) or a pull request to:

- 🧩 **Propose a spec improvement** — a clearer module, a missing failure mode, a better default
- 📐 **Add or refine a template** — and update the matching example so they stay consistent
- 🧪 **Contribute an eval case** — a trigger, recovery, or security scenario worth guarding against
- 📖 **Improve the docs** — sharper wording, better routing, more worked examples

See [CONTRIBUTING.md](CONTRIBUTING.md) for the workflow and the public-release boundary.

---

## 🙏 Acknowledgments

> This specification is distilled from real production practice and shaped by the Agent tooling ecosystem it targets.

| Ecosystem | Role |
|---|---|
| **Claude Code** | Skill entry, frontmatter, and workflow conventions the spec builds on |
| **Codex** | Cross-client Agent execution the spec stays compatible with |
| **Agent Skills authors** | The community practice of packaging repeatable workflows |

The public edition abstracts an internal full-lifecycle standard into a reusable form — keeping only what is explainable, auditable, and safe to distribute, with no private paths, credentials, or customer data.

---

## 👋 About AI Workflow Pro

> 🚀 **AI Workflow Pro** is a one-person product studio building and open-sourcing practical AI workflows — turning AI tools into systems that keep running.

We focus on how AI coding tools (Claude Code / Codex), agents, and automation actually help people ship real work, and we publish hands-on specs, Skills, and field notes along the way.

| Channel | Link |
|---|---|
| 🌐 **Website** | [aiworkflowpro.com](https://aiworkflowpro.com) |
| 💻 **GitHub** | [@aiworkflowpro](https://github.com/aiworkflowpro) |
| 𝕏 **X / Twitter** | [@aiworkflowprolk](https://x.com/aiworkflowprolk) |

---

## 📄 License

Licensed under the [MIT License](LICENSE) — © 2026 AI Workflow Pro.

<div align="center">

### ⭐ If this spec is useful to you, leave a Star

[![Website](https://img.shields.io/badge/🌐_Website-aiworkflowpro.com-FF6B35?style=for-the-badge)](https://aiworkflowpro.com)
[![X / Twitter](https://img.shields.io/badge/𝕏_Twitter-@aiworkflowprolk-000000?style=for-the-badge)](https://x.com/aiworkflowprolk)
[![GitHub](https://img.shields.io/badge/💻_GitHub-aiworkflowpro-181717?style=for-the-badge)](https://github.com/aiworkflowpro)

> 🎯 **Tools change, workflows don't.**

</div>
