<div align="center">
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/imgs/banner-neo-dark.svg">
  <img src="docs/imgs/banner-neo-light.svg" alt="ZENERA NEO" width="720">
</picture>

[![CI](https://github.com/andreyryabov/ZeneraNeo/actions/workflows/ci.yml/badge.svg)](https://github.com/andreyryabov/ZeneraNeo/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Node](https://img.shields.io/badge/node-%E2%89%A524-brightgreen.svg)](https://nodejs.org)
[![OpenAI · Anthropic · Gemini · OpenRouter · vLLM](https://img.shields.io/badge/models-OpenAI%20%C2%B7%20Anthropic%20%C2%B7%20Gemini%20%C2%B7%20OpenRouter%20%C2%B7%20vLLM-8957e5.svg)](#packages)

</div>

# ZeneraNeo

**A toolkit for creating specialized, self-improving agent systems from a
specification and the knowledge your work already has.**

ZeneraNeo turns a clear description of a job into an agentic system built for
that job. Give it the documents, policies, API schemas, files and integrations
the work depends on; describe the outcome and its constraints; then let the
meta-agent design the specialists, tools, skills, memory, handoffs, retrieval
and execution strategy that fit.

## What is it for?

ZeneraNeo is for recurring work that needs more than one general-purpose chat:
technical investigation, policy review, operations triage, software delivery,
analysis, and workflows that must show their evidence. The result is a project
you can run, inspect, improve, review and share.

## ⚠️ Status: Early stage public preview

> **This is an open-source side project for experimentation and chore work.**
> It is **not** the official Zenera AI Platform, and it carries no support or
> stability promises.

## Getting started

### 1. Install

You need:

- [Node.js](https://nodejs.org) 24 or newer
- [Podman](https://podman.io) - agents run commands in a container

Then install the packages:

```sh
npm i -g @zenera/cli @zenera/neo @zenera/rag @zenera/faker
```

### 2. Add a model key and create a project

```sh
zen key add openai
zen init my-project
zen open my-project
```

### 3. Describe the system and generate it

`zen open my-project` opens the project in your default editor, where you can
start editing `SPECIFICATION.md`. 

`SPECIFICATION.md` is file is where most of the work on your
project happens. Use it to describe the purpose of the project, its models, its agents, and other details.

**The purpose of the project:**

```markdown
## Purpose

Answer questions about the Hugging Face Transformers documentation from an index of it, quoting the passage the answer came from.
```

**The models you want the project to use:**

```markdown
## Models

- `openai:gpt-5.6-luna` — one model for the whole project.
- `openai:text-embedding-3-small` — the vectors, for the index and memory both.
```

**The setup that must be done before the first run:**

```markdown
## Setup

Download `https://raw.githubusercontent.com/andreyryabov/ZeneraNeo/main/examples/projects/common/markdown-documentation-transformers.zip`
into `.tmp/`, unzip it, create an index and store it in `assets/docs-db`. It skips when that index
can already answer, and re-embeds it from its own `sources/` when the index is present but its
vectors are not.
```

**What the agents should do:**

```markdown
## Agents

`default` — the only agent.

- Hugging Face documentation assistant.
- The index is the only source. It lists the documents once before its first search, and never answers from its own recollection of the library.
- Anything outside the Hugging Face documentation — another library, the weather, a code review — is declined outright, with a line on what it does cover.
- Every answer names the document and the paragraph it came from.
- Recalls before answering, and commits what it found, so the same question is not searched twice.
- Enable all file tools.
```

**Knowledge - what the agents can read.** 

These are existing materials that do
not change between runs, such as documentation indexes and API schemas. Agents
read them but never change them:

```markdown
## Knowledge

- `assets/docs-db` — the Transformers documentation, read-only at `/assets`.
- create and preload a skill that explains how to do search.
```

**Memory - what the agents should remember.** 

This is what the agents learn
while they work, kept between runs so the same work is not done twice. Describe
what to save and when to look in memory first:

```markdown
## Memory

- use workspace files to store memory files for large data chunks.
- always try to answer from memory first if all data is in memory.
- always save results that took some work to obtain.
- always store detailed sources of information in memory files.
- analyze what queries were made to the memory.
    - if nothing was found and you had to do some work to obtain the result, save it to memory.
```

A complete example of this file is
[`SPECIFICATION.md`](examples/projects/01-docs-rag/SPECIFICATION.md) in
example 01.

Once `SPECIFICATION.md` is filled in, run this in the CLI:

```sh
zen meta run my-project /spec-sync-project
```

It generates and updates the system.

### 4. Answer the meta-agent's questions

Check whether `SPECIFICATION-FEEDBACK.md` exists. 

> The meta-agent does not guess: if something is unclear, it asks. 

- This file is a questionnaire.
- Every question carries two to four written-out answers, each with a `[ ]` box.
- Put an `x` in the box of the answer you want, delete the ones you do not, or
write your own in their place. Any of these counts as an answer. 

Then apply the feedback:

```sh
zen meta run my-project /spec-apply-feedback
```

### 5. Check and run

Finally you can use these commands:

- `zen check` to verify that the generated system is ready to work.
- `zen run` to run it.
- `zen inspect` to examine the resulting trajectory and improvement
  opportunities.

Complete project examples are in [examples/projects](examples/projects/01-docs-rag/README.md).

## Our principles - autonomous specialization and improvement

People define the job; they should not have to hand-design every agent, prompt,
handoff, tool, memory rule and evaluation loop required to do it well.
`SPECIFICATION.md` is the source of intent: the work to perform, the knowledge
to use, the boundaries to keep, and the criteria that define success.

ZeneraNeo's meta-agent turns that intent into a working system, then autonomously
tests it, examines its observable trajectory, and proposes improvements to the
specification. The next system is generated from that improved intent. This is a
continuous loop for making a specialized system more effective without losing
the human-defined purpose and constraints.

![The continuous improvement loop from knowledge and specification to an agentic system, run analysis, and specification updates](docs/imgs/continuous_improvement_loop.svg)

The loop is deliberate. A run is not only an answer; it is evidence about the
system that produced it. ZeneraNeo records the trajectory, evaluates the result
against the specification, identifies changes that improve quality, accuracy,
latency or token use, and turns those findings into reviewable updates.

Projects are ordinary Markdown and YAML, so the intent, generated system and
proposed changes can be versioned in Git and reviewed like any other software.
Each run leaves an inspectable trajectory of the prompts, agent handoffs, tool
calls, results and usage that produced it. Teams can examine that record
themselves or give it to a meta-agent to evaluate outcomes, diagnose weak steps
and propose the next improvement with evidence.

## What the toolkit creates

- **Specialized agents** with distinct responsibilities, models and execution
  boundaries, instead of one prompt asked to do everything.
- **Teams that coordinate** through explicit handoffs, parallel work and
  strategies matched to the task.
- **Knowledge systems** over documentation and API schemas, with retrieval that
  can discover, inspect and verify evidence.
- **Tools and integrations** selected for the job, with file access, sandboxed
  commands and generated integrations where they are needed.
- **Durable memory** that carries useful context between runs without relying on
  an ever-growing chat transcript.
- **Evaluation and observability** through self-tests, success criteria and
  trajectories that explain what happened, how long it took and what it used.
- **Shareable projects** expressed as Markdown and YAML, ready for source
  control, code review and reuse by another team.

## Packages

| Package                                     | Role                                                                                                                                           |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [`@zenera/cli`](packages/cli/README.md)     | The `zen` command-line workflow for creating projects, generating agent systems, running them, testing them and inspecting their trajectories. |
| [`@zenera/neo`](packages/neo/README.md)     | The TypeScript runtime for applications that need agents, models, tools, skills, memory, trajectories and project loading directly in code.    |
| [`@zenera/rag`](packages/rag/README.md)     | Advanced retrieval for agent knowledge: semantic, full-text and hybrid search over documents, plus graph-aware API schema indexing and search. |
| [`@zenera/faker`](packages/faker/README.md) | Mock API generation from OpenAPI and Swagger descriptions, with generated behavior validated against the API contract.                         |

`@zenera/cli` installs `zen`; `@zenera/rag` and `@zenera/faker` extend it with
their own subcommands. Install only the capabilities a project needs.

## Learn more

- [Specification-driven projects](docs/specification.md)
- [Agent configuration](docs/agents-yaml.md)
- [Projects and sharing](docs/projects.md)
- [Agent knowledge and document retrieval](docs/knowledge.md)
- [API integrations](docs/integrations.md)
- [Memory](docs/memory.md)

Early days and moving fast - issues, questions and pull requests are welcome.
[MIT](LICENSE).
