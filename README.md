# Galaxio Gatling Pro

A skill for writing Gatling JVM performance tests in a Galaxio-style repository layout. Works with **Claude Code** (CLI) and **Claude** (desktop app / Cowork).

## What It Covers

- Gatling 3.x with Scala 2.13, Java, and Kotlin
- sbt, Maven, and Gradle build-tool layouts and source roots
- Picatinny config and feeder helpers where available
- HTTP, JDBC, JMS, Kafka, and AMQP test flows
- Open and closed workload models
- Smoke and debug simulations
- Scalafmt-friendly Scala code structure

---

## Installation

### Claude Code (CLI)

Claude Code loads slash commands from `.claude/commands/` in the current project or from `~/.claude/commands/` globally.

**Option A — project-local (one repo)**

Copy the command file into the target performance test repository:

```bash
mkdir -p .claude/commands
curl -sSL https://raw.githubusercontent.com/galaxio/galaxio-gatling-pro/main/.claude/commands/galaxio-gatling-pro.md \
  -o .claude/commands/galaxio-gatling-pro.md
```

**Option B — global (all projects)**

```bash
mkdir -p ~/.claude/commands
curl -sSL https://raw.githubusercontent.com/galaxio/galaxio-gatling-pro/main/.claude/commands/galaxio-gatling-pro.md \
  -o ~/.claude/commands/galaxio-gatling-pro.md
```

**Option C — clone and symlink**

```bash
git clone https://github.com/galaxio/galaxio-gatling-pro.git ~/.claude/skills/galaxio-gatling-pro
ln -s ~/.claude/skills/galaxio-gatling-pro/.claude/commands/galaxio-gatling-pro.md \
      ~/.claude/commands/galaxio-gatling-pro.md
```

This way you can `git pull` to get updates.

---

### Claude (desktop app / Cowork)

The repository is structured as a Claude plugin with a `.claude-plugin/plugin.json` manifest.

**Install via Settings → Plugins → Install from GitHub:**

```
https://github.com/galaxio/galaxio-gatling-pro
```

Claude will detect `.claude-plugin/plugin.json` and register the `galaxio-gatling-pro` skill automatically.

---

## Usage

### Claude Code

After installation the skill is available as a slash command:

```
/galaxio-gatling-pro create a StabilitySimulation for HTTP with Kafka feeder
/galaxio-gatling-pro review src/test/scala/org/galaxio/performance/scenarios/MainScenario.scala
/galaxio-gatling-pro add a DebugSimulation to the existing sbt project
```

Or just describe your task naturally — the skill activates automatically when working on `.scala`, `.java`, `.kt`, `build.sbt`, `pom.xml`, or `build.gradle` files in a Gatling project.

### Claude (desktop app / Cowork)

Once the plugin is installed, use the skill from any conversation:

```
/galaxio-gatling-pro create a closed-model pacing simulation with JDBC feeder
```

Or describe the task in natural language — Claude will apply the skill when it detects Gatling / Galaxio context.

---

## Repository Layout

```text
.claude-plugin/
  plugin.json                         # Claude desktop plugin manifest
.claude/
  commands/
    galaxio-gatling-pro.md            # Claude Code slash command
skills/
  galaxio-gatling-pro/
    SKILL.md                          # Full skill definition (Claude desktop)
agents/
  openai.yaml                         # OpenAI Codex agent definition (legacy)
README.md
LICENSE
```

---

## Skill Scope

The skill generates and reviews code within the Galaxio layer boundaries:

- `cases/` — atomic HTTP / Kafka / JDBC / AMQP / JMS actions
- `feeders/` — reusable feeder logic (prefer Picatinny feeders)
- `scenarios/` — business flows composed from cases
- `*Simulation` classes — injection profiles and protocols only

NFR assertions and pass/fail gates are opt-in and only added when explicitly requested.

---

## License

Apache 2.0
