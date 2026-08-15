+++
title = "Before an AI Coding Agent Touches Your Hiero Project: A Security Preflight"
date = 2026-08-15
draft = false
featured_image = "/images/Hiero_v4.png"
categories = ["Blog"]
tags = ["Security", "AI Agents", "Developer Experience", "Open Source"]
duration = "7 min read"
abstract = "A practical checklist for limiting agent permissions, protecting credentials, reviewing executable surfaces, and preserving evidence when using AI coding agents in Hiero projects."
slug = "ai-coding-agent-security-preflight"

[[authors]]
name = "Michael Kantor"
title = "President"
organization = "Hashgraph Online"
link = "https://github.com/kantorcodes"
+++

AI coding agents can shorten the path from an issue to a working pull request. They can also read files, edit source code, install packages, run commands, change workflows, and connect to external tools. That makes them useful, but it also means they should be treated as privileged automation rather than as autocomplete.

This preflight is for contributors using coding agents in Hiero projects. It is intentionally tool-agnostic. The goal is not to prevent agents from acting. The goal is to make their authority explicit, keep high-risk operations reviewable, and leave enough evidence for another maintainer to understand what happened.

## Start With a Concrete Threat Model

Before opening an agent session, identify the failures that matter for the task. Common examples include:

- a destructive shell or Git command running with broader scope than intended;
- a testnet or developer credential being read, printed, or committed;
- a package install executing an unexpected lifecycle script;
- repository content, an issue, or tool output steering the agent away from the requested task;
- an MCP server, hook, skill, or plugin gaining permissions that were never reviewed;
- a workflow or release file changing as a side effect of unrelated work;
- generated code passing a narrow test while violating repository architecture or contribution rules.

The right controls depend on the repository and task. A documentation correction does not need the same authority as a dependency migration or consensus-node change.

## 1. Isolate the Work

Begin from a clean branch or worktree. Do not give an agent a working directory that already contains unrelated edits.

```bash
git status --short
git switch -c agent/<short-task-name>
```

For tasks that can interact with a network:

- use testnet or a local environment;
- use a dedicated account with the minimum required balance and permissions;
- avoid production credentials entirely;
- scope API tokens to the repository and operations the task actually needs;
- remove credentials from the environment when the relevant step is complete.

A coding agent should not inherit every credential available in a developer's normal shell simply because those credentials already exist.

## 2. Define the Approval Boundary Before the First Command

Decide which operations may proceed normally and which require explicit review. A practical default is to allow repository reads and routine tests, while pausing before:

- package installation or lockfile regeneration;
- network requests outside the task's documented dependencies;
- writes to CI, release, deployment, or security-sensitive configuration;
- changes to agent hooks, MCP servers, skills, plugins, or editor configuration;
- reads of credential-bearing files;
- destructive filesystem or Git operations;
- force pushes, tag creation, publishing, or release commands;
- blockchain transactions, even on testnet, unless the exact action is part of the task.

The important point is not the exact list. The important point is that the boundary exists before the agent encounters a tempting shortcut.

## 3. Review Executable Surfaces Before Running Them

Repository content is data until you decide to execute it. Review the files that can introduce behavior before asking an agent to install or run the project:

- package manifests and lockfiles;
- install, prepare, postinstall, and other lifecycle scripts;
- shell scripts and task runners;
- CI and release workflows;
- container definitions;
- MCP server declarations;
- agent hooks, rules, skills, and plugins;
- generated code that opens network connections or spawns subprocesses.

Do not run an unfamiliar bootstrap script merely because a README or tool response says it is required. Inspect it first, confirm why it is needed, and prefer the repository's documented contributor path.

## 4. Add a Static Preflight

A static review cannot prove that a repository is safe, but it can expose obvious risk before code is executed. At minimum, inspect the planned diff and high-risk file classes:

```bash
git diff --check
git diff --stat
git diff -- .github package.json pnpm-lock.yaml yarn.lock uv.lock pyproject.toml
```

Adapt those paths to the repository's language and build system.

Teams can also use a dedicated scanner. One open-source option is [HOL Guard](https://hol.org/guard/security), which includes a static scanner for agent skills, MCP servers, plugins, and repository security surfaces. It can be run ephemerally without adding it to the project's dependency graph:

```bash
uvx --from hol-guard plugin-scanner scan .
```

That command is a review signal, not a verdict. A clean result does not replace source review, repository-specific tests, or maintainer judgment.

## 5. Keep High-Risk Actions Small and Attributable

When an agent needs broader authority, split the work into checkpoints. For example:

1. inspect the dependency change;
2. approve the package operation;
3. review the resulting manifest and lockfile diff;
4. run tests;
5. approve any network or publishing step separately.

This is safer than granting a broad instruction such as "fix everything and publish it." It also makes failures easier to diagnose because each side effect has a clear purpose.

For blockchain-related development, record the network, account, transaction type, and expected effect before approval. A human should be able to compare the intended action with the actual transaction before value or state changes.

## 6. Preserve Useful Evidence

A good agent-assisted pull request should be understandable without access to the original chat session. Preserve evidence that helps reviewers reproduce and evaluate the work:

- the issue or task being addressed;
- the files and behavior changed;
- commands that altered dependencies, configuration, or generated artifacts;
- the tests, linters, and builds that ran;
- any security findings and how they were resolved;
- any testnet transaction identifiers relevant to the change;
- known limitations or work that remains.

Avoid copying raw prompts or logs that contain secrets or unrelated local paths. Evidence should increase reviewability without creating a second data-leak problem.

## 7. Review the Result Like an Untrusted Contribution

AI-generated code should receive the same review as code from an unfamiliar contributor. Read the diff in risk order:

1. credentials, network, and permission changes;
2. CI, release, and deployment files;
3. dependency and lockfile changes;
4. agent, MCP, hook, and plugin configuration;
5. source code and tests;
6. generated files and documentation.

Then run the repository's documented validation commands. For the Hiero website, for example, the contributor workflow requires:

```bash
pnpm lint
pnpm build
```

Other Hiero repositories have different build and test requirements. Follow the repository's own contribution documentation rather than assuming one command set applies everywhere.

## A Minimal Preflight Checklist

Before allowing an agent to act:

- [ ] The task and allowed scope are written down.
- [ ] The branch or worktree is clean and isolated.
- [ ] Only testnet, local, or least-privilege credentials are available.
- [ ] High-risk commands require explicit review.
- [ ] Package scripts, workflows, hooks, MCP servers, skills, and plugins were inspected before execution.
- [ ] Dependency and configuration changes are reviewed separately from source changes.
- [ ] Repository-specific lint, test, and build commands pass.
- [ ] The final diff contains no unrelated changes.
- [ ] The pull request records the evidence another maintainer needs.

## The Practical Standard

AI coding agents are most valuable when they can take meaningful action. The answer is not to remove all authority. It is to grant the smallest useful authority, make expansion visible, and preserve enough evidence to review the outcome.

That approach lets contributors move quickly without asking maintainers to trust an opaque sequence of prompts, tools, and side effects.

_Disclosure: I am President of Hashgraph Online, which maintains HOL Guard._
