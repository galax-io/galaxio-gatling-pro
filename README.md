# Galaxio Gatling Pro

> **⚠️ DEPRECATED — this repository has moved.**
>
> The skill now ships as the `galaxio-gatling-pro` plugin in the
> [Galaxio Performance Kit marketplace](https://github.com/galax-io/ai-plugins), where it installs
> in Cursor, Claude Code and Codex from one source. This repository is archived and receives no
> further updates. See [Installation](#installation) for the new commands.

[![Deprecated](https://img.shields.io/badge/status-deprecated-red.svg)](https://github.com/galax-io/ai-plugins)
[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
![Agent Skill](https://img.shields.io/badge/type-agent--skill-orange.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet.svg)
![Codex](https://img.shields.io/badge/Codex-compatible-green.svg)
[![CI](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml/badge.svg)](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml)

Agent skill for writing, reviewing, and refactoring Gatling JVM performance tests in Galaxio-style repository layouts.

## Installation

Install from the [Galaxio Performance Kit marketplace](https://github.com/galax-io/ai-plugins).
The `git clone` and `npx skills add` recipes that used to live here are no longer maintained: this
repository is frozen at the last standalone release, and everything new lands in the marketplace.

### Claude Code

```bash
claude plugin marketplace add galax-io/ai-plugins
```

```text
/plugin install galaxio-gatling-pro@galaxio-performance-kit
```

### OpenAI Codex

```bash
codex plugin marketplace add galax-io/ai-plugins
```

Then invoke the skill with `$galaxio-gatling-pro`.

### Cursor

Copy `plugins/galaxio-gatling-pro` from the marketplace repository into `~/.cursor/plugins/local`
and restart Cursor, until the kit is published to the Cursor marketplace.

## What It Covers

- Gatling 3.x with Scala, Java, and Kotlin
- sbt, Maven, and Gradle build-tool layouts
- Picatinny-first config and feeder helpers
- HTTP, JDBC, JMS, Kafka, and AMQP protocols
- Open and closed workload models
- Smoke and debug simulations
- Build-tool-correct source roots and run commands
- Scalafmt-friendly Scala code generation

## Activation Triggers

| Signal | Examples |
|--------|----------|
| Keywords | gatling, performance testing, load testing, simulation |
| File patterns | `*.scala`, `*.java`, `*.kt`, `build.sbt`, `pom.xml`, `build.gradle`, `gatling.conf` |
| Context | Creating, reviewing, or refactoring Gatling performance tests |

## Repository Layout

```text
SKILL.md              # Full skill content (817 lines of guidance)
CLAUDE.md             # Claude Code discovery file
manifest.json         # Machine-readable activation metadata
agents/openai.yaml    # Codex UI integration
.markdownlint.json    # Lint configuration
.github/workflows/    # CI: lint + skill validation
README.md             # This file
LICENSE               # Apache 2.0
```

## CI

- **Lint** — markdownlint on all `.md` files
- **Validate** — checks SKILL.md frontmatter, required files, valid JSON/YAML
- **Link check** — validates internal markdown links

## License

[Apache 2.0](./LICENSE)
