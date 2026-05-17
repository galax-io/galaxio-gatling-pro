# Galaxio Gatling Pro

[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
![Agent Skill](https://img.shields.io/badge/type-agent--skill-orange.svg)
![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-blueviolet.svg)
![Codex](https://img.shields.io/badge/Codex-compatible-green.svg)
[![CI](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml/badge.svg)](https://github.com/galax-io/galaxio-gatling-pro/actions/workflows/lint.yml)

Agent skill for writing, reviewing, and refactoring Gatling JVM performance tests in Galaxio-style repository layouts.

## Installation

### Quick Install (npx)

Global:

```bash
npx skills add galax-io/galaxio-gatling-pro -g
```

Project-local:

```bash
npx skills add galax-io/galaxio-gatling-pro
```

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

Add as AGENTS.md reference or clone into workspace. Codex auto-discovers via `agents/openai.yaml` and `manifest.json`:

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git
```

Then reference in your `AGENTS.md`:

```markdown
## Skills

- [galaxio-gatling-pro](./galaxio-gatling-pro/SKILL.md) — Gatling JVM performance test patterns
```

### Marketplace

Published as a reusable agent skill. To add to your project from the marketplace:

1. Search for `galaxio-gatling-pro` in your agent marketplace
2. Click **Install** or add manually via git clone above
3. Skill auto-activates on Gatling-related file patterns

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
