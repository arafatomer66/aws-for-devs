# Amazon Q Developer (formerly CodeWhisperer)

**TL;DR** — AWS's AI coding assistant. Auto-complete, inline chat, transform legacy code, generate tests. Free tier for individuals.

## What it is

An IDE plugin (VS Code, JetBrains, Visual Studio, Eclipse, AWS Toolkit) that suggests code as you type, answers questions about your codebase, and runs multi-file refactors. Integrates with AWS docs and your account context.

## Features

- **Inline completion** — AI auto-complete.
- **Chat** — ask questions about code / AWS.
- **Code transformations** — Java 8 → Java 17/21, .NET Framework → modern .NET.
- **Test generation.**
- **Security scans** — find vulnerabilities, suggest fixes.
- **Amazon Q for Business** — RAG over your enterprise data sources (different SKU).
- **Amazon Q in your IDE / CLI / Slack** — integrations.

## Pricing

- **Free tier (individuals)** — generous, suitable for learning/small projects.
- **Pro tier:** $19/user/mo.

## Q Developer vs Claude Code / Cursor / Copilot

- Q Developer is AWS-flavored — strong on AWS APIs, CDK, IAM.
- Copilot / Claude Code / Cursor are more general.
- Many devs use both for different strengths.

## Real-world example

> Java team upgrading 200k LoC from Java 8 to 21:
> - Run Q Developer's "Code Transformation" feature.
> - Reviews + commits the suggested PRs.
> - Saves weeks of manual work.

## Gotchas

- **Context window matters.** Q reads only the open files unless using project-aware features.
- **Suggestions are suggestions** — always review (especially IAM and SQL).
- **Code references** flag possible OSS license matches.

## Related

- [Bedrock](./bedrock.md)
