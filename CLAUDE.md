# Galaxio Gatling Pro

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

Global (all projects):

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git ~/.claude/skills/galaxio-gatling-pro
```

Project-local:

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git .claude/skills/galaxio-gatling-pro
```

## Install — OpenAI Codex

Add as workspace skill. Codex auto-discovers via `agents/openai.yaml` and `manifest.json`:

```bash
git clone https://github.com/galax-io/galaxio-gatling-pro.git .codex/skills/galaxio-gatling-pro
```
