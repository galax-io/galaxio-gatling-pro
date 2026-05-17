# Galaxio Gatling Pro

[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Agent Skill](https://img.shields.io/badge/type-agent--skill-orange.svg)]()
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet.svg)]()
[![Codex](https://img.shields.io/badge/Codex-compatible-green.svg)]()
[![CI](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml/badge.svg)](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml)

Agent skill for writing, reviewing, and refactoring Gatling JVM performance tests in Galaxio-style repository layouts.

## Installation

### Claude Code

Global (available in all projects):

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git ~/.claude/skills/galaxio-gatling-pro
```

Project-local (current project only):

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git .claude/skills/galaxio-gatling-pro
```

### OpenAI Codex

Clone into workspace. Codex auto-discovers via `agents/openai.yaml` and `manifest.json`:

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git .codex/skills/galaxio-gatling-pro
```

Or reference directly in your Codex skill configuration.

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
.github/workflows/    # CI: lint + skill validation + benchmark
README.md             # This file
LICENSE               # Apache 2.0
```

## CI

- **Lint** — markdownlint on all `.md` files
- **Validate** — checks SKILL.md frontmatter, required files, valid JSON/YAML
- **Benchmark** — skill quality metrics (weekly + manual dispatch)

## License

[Apache 2.0](./LICENSE)
