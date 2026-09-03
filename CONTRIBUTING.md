# Contributing Guide for informatica-claude-plugins

This page lists the operational governance model of this project, as well as the recommendations and requirements for how to best contribute to `informatica-claude-plugins`. As always, thanks for contributing — we hope these guidelines make it easier and shed some light on our approach and processes.

## Governance Model

### Salesforce Sponsored

The intent and goal of open sourcing this project is to increase the contributor and user base. However, only Salesforce employees will be given `admin` rights and will be the final arbiters of what contributions are accepted or not. See [CODEOWNERS](CODEOWNERS) for the current maintainer.

## Getting started

There is no dedicated Slack channel, mailing list, or roadmap document for this project today — use [GitHub Issues](https://github.com/forcedotcom/informatica-claude-plugins/issues) on this repo for all discussion, questions, and proposals.

This repo is a container for Claude Code plugins targeting Informatica IDMC (Intelligent Data Governance and Catalog / Claire). See the root [`README.md`](README.md) and [`CLAUDE.md`](CLAUDE.md) for the current plugin inventory and architecture notes before contributing.

## Issues, requests & ideas

Use the GitHub Issues page to submit issues, enhancement requests, and discuss ideas.

### Bug Reports and Fixes
- If you find a bug, please search for it in [Issues](https://github.com/forcedotcom/informatica-claude-plugins/issues), and if it isn't already tracked, [create a new issue](https://github.com/forcedotcom/informatica-claude-plugins/issues/new). Even if an issue is closed, feel free to comment and add details — it will still be reviewed.
- Issues that have already been identified as a bug (i.e., reproducible) will be labeled `bug`.
- If you'd like to submit a fix for a bug, [send a pull request](#creating-a-pull-request) and mention the issue number. Since this repo has no automated test suite (see below), describe how you validated the fix manually.

### New Features / New Plugins
- If you'd like to propose a new plugin, or new capability inside the existing `informatica-in-claude/` plugin, describe the problem you want to solve in a [new issue](https://github.com/forcedotcom/informatica-claude-plugins/issues/new) first.
- Issues identified as a feature request will be labeled `enhancement`.
- Please wait for feedback from the maintainer before spending significant time writing the code — an `enhancement` may not align with the project's current direction.

**Adding a new plugin** should follow the same shape as `informatica-in-claude/` (see the root [`CLAUDE.md`](CLAUDE.md) for full detail):
- A top-level directory named for the plugin.
- Its own `.claude-plugin/plugin.json` manifest.
- Its own `.mcp.json` MCP server config, if the plugin needs one.
- `skills/*/SKILL.md` for each skill the plugin ships.
- Its own `CLAUDE.md`/`README.md` describing what the plugin does and its current limitations.
- If this repo is ever turned into an installable marketplace, a root `.claude-plugin/marketplace.json` listing each plugin directory will also be needed — that file does not exist yet.

### Tests, Documentation, Miscellaneous
- If you'd like to improve the docs, clarify a skill's instructions, or propose an alternative structure, we'd be happy to hear about it.
- Trivial changes can go straight to a [pull request](#creating-a-pull-request). Larger changes should start as an issue to discuss the approach first.

## Contribution Checklist

- [x] Clean, minimal, well-formatted JSON/Markdown — this repo has no code, only plugin manifests, MCP configs, and skill docs.
- [x] Commits should be atomic and messages should be descriptive. Reference related issue numbers where relevant.
- [x] **No build/test/lint tooling exists in this repo** (no `package.json`, `Makefile`, `scripts/`, or CI workflows). Validate changes by manually reading the JSON and Markdown you touched — confirm `plugin.json`/`.mcp.json` are valid JSON, and that any `SKILL.md` you add or edit follows the shape of existing skills.
- [x] Reviews: changes must be approved via peer code review before merging.

## Creating a Pull Request

1. **Ensure the bug/feature was not already reported** by searching [Issues](https://github.com/forcedotcom/informatica-claude-plugins/issues). If none exists, create a new issue so other contributors can track what you're working on.
2. **Fork** the repo (or branch directly if you have write access).
3. **Create** a new branch for your work (e.g. `git checkout -b fix-issue-11`).
4. **Commit** changes to your branch.
5. **Push** your branch to your fork or to this repo.
6. **Submit** a pull request against the `main` branch, referencing the issue(s) it addresses. Keep the PR focused — avoid bundling unrelated changes.
7. **Sign** the Salesforce CLA if prompted (see below).

> **Note:** Be sure to [sync your fork](https://help.github.com/articles/syncing-a-fork/) before opening a pull request.

## Contributor License Agreement ("CLA")

In order to accept your pull request, we need you to submit a CLA. You only need to do this once to work on any of Salesforce's open source projects.

Complete your CLA here: <https://cla.salesforce.com/sign-cla>

## Code of Conduct

Please follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## License

By contributing your code, you agree to license your contribution under the terms of this project's [LICENSE](LICENSE) (Apache License 2.0) and to sign the [Salesforce CLA](https://cla.salesforce.com/sign-cla).
