# Galaxio Gatling Pro

> **⚠️ DEPRECATED — this repository has moved.**
>
> The skill now ships as the `galaxio-gatling-pro` plugin in the
> [Galaxio Performance Kit marketplace](https://github.com/galax-io/ai-plugins). This repository is
> archived and receives no further updates.

Agent skill for Gatling JVM performance testing in Galaxio-style repositories.

## Skill Entry Point

Full skill content: [SKILL.md](./SKILL.md)

## Activation

Activates when working with:

- Gatling 3.x performance test projects (Scala, Java, Kotlin)
- Build tools: sbt, Maven, Gradle
- File patterns: `*.scala`, `*.java`, `*.kt`, `build.sbt`, `pom.xml`, `build.gradle`, `gatling.conf`
- Keywords: gatling, performance testing, load testing, simulation

## Install — Claude Code

```bash
claude plugin marketplace add galax-io/ai-plugins
```

```text
/plugin install galaxio-gatling-pro@galaxio-performance-kit
```

## Install — OpenAI Codex

```bash
codex plugin marketplace add galax-io/ai-plugins
```

Then invoke the skill with `$galaxio-gatling-pro`.
