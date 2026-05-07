# C/C++test AI Agent Skills

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Skills overview](#skills-overview)
- [Integrating skills into your project](#integrating-skills-into-your-project)
- [Basic configuration](#basic-configuration)
- [Additional configuration for GitHub CI/CD](#additional-configuration-for-github-cicd)
- [Using skills with C/C++test Professional](#using-skills-with-cctest-professional)
- [Using skills with Windows PowerShell](#using-skills-with-windows-powershell)
- [References](#references)

## Overview

This repository contains C/C++test static analysis skills for AI coding agents, enabling you to fix static analysis violations both autonomously in CI/CD and interactively in a local environment. 

A pre-configured demo project is available at https://github.com/parasoft/cpptest-ai-agent-demo.

## Prerequisites

- AI coding agent installed and configured
- C/C++test installed and configured
- C/C++test's MCP server registered
- GitHub CLI installed - for using the GitHub CI/CD pipeline

## Skills overview

1. **`cpptest-fix-all-violations`** — Runs C/C++test static analysis and iteratively fixes all reported violations, rule by rule. After fixing each rule, the fixes are verified. Supports configurable iteration and rule limits, and can optionally commit fixes after each rule. Analysis is performed by executing the `cpptest-analyze.sh` script.

   How to use: the primary use case is autonomous fixing inside a CI/CD pipeline (see [Additional configuration for GitHub CI/CD](#additional-configuration-for-github-cicd) below). The skill can also be used locally by prompting the AI agent, e.g. *"Fix all violations in this project"* or *"Fix all MISRA violations"*. To fix a single, already-known violation, use `cpptest-fix-one-violation` instead.

   Configuration environment variables:

   - `CPPTEST_COMMIT_FIXES` — If set to `true` or `1`, commit fixes after each rule. Default: not set.
   - `CPPTEST_ITERATION_LIMIT` — Maximum number of fix-and-verify attempts per rule before declaring it unfixable. Default: `3`.
   - `CPPTEST_RULES_LIMIT` — Maximum number of rules to process before stopping. Default: `20`.

2. **`cpptest-fix-one-violation`** — Fixes a single, already-known C/C++test static analysis violation described by the user. Does not run analysis upfront — violation details are taken directly from the user's prompt. Verification is only performed when explicitly requested, and is performed by executing the `cpptest-analyze.sh` script.

   How to use: prompt the AI agent with the details of a specific violation, e.g. *"Fix and verify CERT_C-DCL00-a: Declare local variable 'sensor' as const at sensor.c:55"*. To fix multiple or all violations, use `cpptest-fix-all-violations` instead.

## Integrating skills into your project

Assuming the project is located at `<PROJECT_DIR>`.

1. Copy the `skills` folder into your project — typically into `<PROJECT_DIR>/.agents`. See the agent documentation for the location of skill definition files.
2. Copy the `agents/AGENTS.md` file into `<PROJECT_DIR>`. See the agent documentation for the location of agent instruction files.
3. Copy the `scripts/cpptest-analyze.sh` file into `<PROJECT_DIR>`.
4. To use the GitHub Actions pipeline, copy the `workflows` folder into `<PROJECT_DIR>/.github`.

A typical project layout looks like this:

```
<PROJECT_DIR>
  .agents/skills/
     cpptest-fix-all-violations/SKILL.md
     cpptest-fix-one-violation/SKILL.md
  .github/workflows/
     cpptest-agent-prompt.md
     cpptest-agent-run.sh
     cpptest-autofix-github.yml
  AGENTS.md
  cpptest-analyze.sh
```

## Basic configuration

Review and update the following configuration files:
- `cpptest-analyze.sh` should run the C/C++test static analysis with additional build and verification steps. It is intended to be used by the C/C++test skills, whenever the static analysis needs to be re-executed or applied fixes need to be verified. The skills expect the static analysis report file to be created in `<PROJECT_DIR>/reports/report.xml`; adjust the skill definition files (`SKILL.md`) if needed.

## Additional configuration for GitHub CI/CD

Review and update the following configuration files in `.github/workflows`:
- `cpptest-autofix-github.yml` should execute the build pipeline with C/C++test static analysis and automatically fix the reported static analysis violations. Before fixing the violations, a separate branch is created (`autofix/pr-<number>/<timestamp>`). After violations are fixed, the changes are committed (via `CPPTEST_COMMIT_FIXES=1`), the autofix branch is pushed to the remote repository, and a pull request is created automatically.
- `cpptest-agent-run.sh` should configure and run an AI agent with the prompt defined in `cpptest-agent-prompt.md`. Adjust to your specific AI coding agent.

See the `Run C/C++test AI Autofix` step in `cpptest-autofix-github.yml` for details.

> **Note:** To allow GitHub Actions to create pull requests automatically, enable the following option in your repository: **Settings > Actions > General > Allow GitHub Actions to create and approve pull requests**.

## Using skills with C/C++test Professional

The integration scripts and workflow in this project are configured for C/C++test Standard. To use them with C/C++test Professional, review and update both the local analysis command (`cpptest-analyze.sh`) and the GitHub workflow configuration (`cpptest-autofix-github.yml`) so they match your installation and project layout.

Update `cpptest-analyze.sh` so its `cpptestcli` command invocation matches your C/C++test Professional setup. In particular, verify the workspace location, test configuration, report directory, and analysis input file. For example:
```bash
cpptestcli -data ../workspace -config "builtin://Recommended Rules" -report "reports" -bdf "cpptestscan.bdf"
```

If you use the GitHub CI/CD pipeline, review the `Run C/C++test` step in `cpptest-autofix-github.yml` and adjust the `run-cpptest-action` inputs for C/C++test Professional. This typically includes settings such as the analysis input, workspace location, and any additional parameters required by your environment. See [Customizing the Action to Run C/C++test Professional](https://github.com/parasoft/run-cpptest-action#customizing-the-action-to-run-cctest-professional) for details.

## Using skills with Windows PowerShell

By default, the skills are prepared to work with a `bash` shell. To use the skills in a Windows-based environment with PowerShell, the `cpptest-analyze.sh` and `cpptest-agent-run.sh` scripts must be converted to PowerShell scripts (`cpptest-analyze.ps1` and `cpptest-agent-run.ps1`). Then update the `SKILL.md` files: replace `cpptest-analyze.sh` references with `cpptest-analyze.ps1`, and change the script invocation code block language from `bash` to `powershell`.

Additionally, for the GitHub CI/CD pipeline, review and adjust `cpptest-autofix-github.yml` (replace `cpptest-agent-run.sh` with its `.ps1` counterpart).

## References

- Copilot skills:
    - https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-skills
- Claude Code skills:
    - https://code.claude.com/docs/en/skills
- Codex CLI skills:
    - https://developers.openai.com/codex/skills
- Gemini CLI skills:
    - https://geminicli.com/docs/cli/skills
