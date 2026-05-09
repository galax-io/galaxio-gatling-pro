---
description: Create, review, or refactor Gatling JVM performance tests in Galaxio style (Scala/Java/Kotlin, sbt/Maven/Gradle, Picatinny, HTTP/JDBC/JMS/Kafka/AMQP).
argument-hint: "<file path, simulation class, or describe what to build>"
---

Use when creating, reviewing, or refactoring Gatling JVM performance tests
in Galaxio style: Scala/Java/Kotlin projects with sbt, Maven, or Gradle,
Gatling 3.x, Picatinny config/feeders where available,
cases/feeders/scenarios/simulations layout,
HTTP/JDBC/JMS/Kafka/AMQP protocols, open and closed workload models,
smoke/debug simulations, and build-tool-correct project structure.

$ARGUMENTS

---

# Galaxio Gatling Pro

## Core Rules

Use Gatling `3.11.x` and Scala `2.13.x` as the Galaxio baseline.

Supported JVM combinations:

- Scala + sbt: Scala DSL, `src/test/scala`, optional `src/it/scala`.
- Scala + Maven: Scala DSL, `src/test/scala`, `scala-maven-plugin`.
- Scala + Gradle: Scala DSL, `src/gatling/scala`, Gradle `gatling` source set.
- Java + Maven: Java DSL, `src/test/java`.
- Java + Gradle: Java DSL, `src/gatling/java`, Gradle `gatling` source set.
- Kotlin + Maven: Java DSL from Kotlin, `src/test/kotlin`, `kotlin-maven-plugin`.
- Kotlin + Gradle: Java DSL from Kotlin, `src/gatling/kotlin`, Gradle `gatling` source set.

Prefer `org.galaxio.gatling-picatinny` helpers when the repo has the dependency.
Picatinny examples in this skill are Scala-first; for Java/Kotlin, keep the same
architecture and use small local config/feeder helpers if Picatinny is not exposed
through the project's chosen DSL.

Generated Scala code in sbt projects must pass:

```bash
sbt scalafmtAll scalafmtSbt
```

When changing existing repo, follow local style first. If no style, use this skill.

## Build Tool Matrix

Use the build tool's conventional Gatling source roots. Do not move everything into
`src/test/scala` just because the Galaxio template started as sbt.

| Build tool | Languages | Simulation source root | Resource root | Run one simulation |
| --- | --- | --- | --- | --- |
| sbt | Scala | `src/test/scala` or `src/it/scala` | `src/test/resources` | `sbt 'Gatling/testOnly <fqcn>'` |
| Maven | Scala | `src/test/scala` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Maven | Java | `src/test/java` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Maven | Kotlin | `src/test/kotlin` | `src/test/resources` | `./mvnw gatling:test -Dgatling.simulationClass=<fqcn>` |
| Gradle | Scala | `src/gatling/scala` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |
| Gradle | Java | `src/gatling/java` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |
| Gradle | Kotlin | `src/gatling/kotlin` | `src/gatling/resources` | `./gradlew gatlingRun --simulation <fqcn>` |

Compile/pre-flight commands:

```bash
sbt Gatling/compile
./mvnw test-compile
./gradlew testClasses
```

## Project Layout

Keep the Galaxio boundaries regardless of language or build tool. Only the source
root changes.

Canonical Scala/sbt layout:

```text
src/test/scala/org/galaxio/performance/
  performance.scala # package object with protocols
  cases/            # atomic actions: HTTP, Kafka, JDBC, AMQP, JMS
  feeders/          # custom feeders; prefer Picatinny feeders
  scenarios/        # flows built from cases
  *Simulation.scala # simulations only
src/test/resources/
  simulation.conf
  gatling.conf
  logback.xml
```

Keep boundaries strict:

- `cases`: request/action only. No workload. No scenario.
- `feeders`: data only. No requests.
- `scenarios`: business flow only. No injection profile.
- `simulations`: injection, protocols, max duration. No request definitions.
- `performance.scala`, `Performance.java`, or `Performance.kt`: shared protocols only.

## Imports

Scala DSL base:

```scala
import io.gatling.core.Predef._
import io.gatling.http.Predef._
import org.galaxio.gatling.config.SimulationConfig._
```

Picatinny helpers:

```scala
import org.galaxio.gatling.feeders._
import org.galaxio.gatling.utils.IntensityConverter._
```

Java DSL:

```java
import static io.gatling.javaapi.core.CoreDsl.*;
import static io.gatling.javaapi.http.HttpDsl.*;
import io.gatling.javaapi.core.*;
import io.gatling.javaapi.http.*;
```

Kotlin DSL:

```kotlin
import io.gatling.javaapi.core.CoreDsl.*
import io.gatling.javaapi.http.HttpDsl.*
import io.gatling.javaapi.core.*
import io.gatling.javaapi.http.*
```

## Config

Use Picatinny `SimulationConfig`. Default params: `baseUrl`, `intensity`, `stagesNumber`, `rampDuration`, `stageDuration`, `testDuration`.

Do not hardcode env data. Put host, login, password, topic, queue, DB URL in `simulation.conf` or pass via `-Dparam=value`.

## Cases

Case = one atomic action.

```scala
object HttpCases {
  val getMainPage = http("GET /")
    .get("/")
    .check(status.is(200))
}
```

## Feeders

Prefer Picatinny feeders:

```scala
object Feeders {
  val messageId = RandomUUIDFeeder("messageId")
  val phone     = RandomPhoneFeeder("phone")
}
```

## Scenarios

```scala
object MainScenario {
  def apply(): ScenarioBuilder = new MainScenario().scn
}

class MainScenario {
  val scn: ScenarioBuilder = scenario("Main Scenario")
    .feed(Feeders.messageId)
    .exec(HttpCases.getMainPage)
}
```

## Simulations

```scala
class StabilitySimulation extends Simulation {
  setUp(
    MainScenario().inject(
      rampUsersPerSec(0).to(intensity).during(rampDuration),
      constantUsersPerSec(intensity).during(stageDuration),
    ),
  ).protocols(httpProtocol)
    .maxDuration(testDuration)
}
```

Smoke/debug:

```scala
class DebugSimulation extends Simulation {
  setUp(
    MainScenario().inject(atOnceUsers(1)),
  ).protocols(httpProtocol)
    .maxDuration(1.minute)
}
```

## Assertions And NFR

Do not add NFR/assertions by default. Add only when user explicitly asks.

## Do Not

Do not mix cases, scenario, simulation in one file. Do not put injection in scenarios.
Do not hardcode secrets. Do not use `Thread.sleep`; use `pause` or `pace`.
Do not use throttling as main workload model. Do not add NFR gates unless user asks.
Do not use Scala 3 unless repo already supports it.
Do not omit `scala-maven-plugin` in Maven Scala projects.
