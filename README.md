# Galaxio Gatling Pro

`galaxio-gatling-pro` is a Codex skill for writing Gatling JVM performance tests in a Galaxio-style repository layout.

## What It Covers

- Gatling 3.x
- Scala, Java, and Kotlin Gatling projects
- sbt, Maven, and Gradle build-tool layouts
- Picatinny-first config and feeder helpers where available
- HTTP, JDBC, JMS, Kafka, and AMQP test flows
- Open and closed workload models
- Smoke and debug simulations
- Build-tool-correct source roots and run commands
- Scalafmt-friendly Scala code structure

## Repository Layout

```text
SKILL.md
agents/openai.yaml
README.md
LICENSE
```

## Skill Scope

The skill is meant for:

- `performance/cases` for atomic requests and actions
- `performance/feeders` for reusable feeder logic
- `performance/scenarios` for business flows
- `src/test/...` or `src/gatling/...` simulation layers, adapted to the target build tool conventions

## Notes

- NFR and assertion guidance is opt-in and only used when explicitly requested.
- The default naming and project structure follow the Galaxio public Gatling patterns.
- The repository is Apache 2.0 licensed.

## Usage

Use this repo as the skill source for Codex or copy the skill files into the target workspace structure used by your performance tests.
