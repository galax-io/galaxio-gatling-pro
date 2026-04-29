---
name: galaxio-gatling-pro
description: >
  Use when creating, reviewing, or refactoring Gatling JVM performance tests
  in Galaxio style: Scala/Java/Kotlin projects with sbt, Maven, or Gradle,
  Gatling 3.x, Picatinny config/feeders where available,
  cases/feeders/scenarios/simulations layout,
  HTTP/JDBC/JMS/Kafka/AMQP protocols, open and closed workload models,
  smoke/debug simulations, and build-tool-correct project structure.
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

Same layout adapted to each build tool:

```text
# sbt or Maven + Scala
src/test/scala/org/galaxio/performance/{performance.scala,cases,feeders,scenarios,*Simulation.scala}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Maven + Java
src/test/java/org/galaxio/performance/{Performance.java,cases,feeders,scenarios,*Simulation.java}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Maven + Kotlin
src/test/kotlin/org/galaxio/performance/{Performance.kt,cases,feeders,scenarios,*Simulation.kt}
src/test/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Scala
src/gatling/scala/org/galaxio/performance/{performance.scala,cases,feeders,scenarios,*Simulation.scala}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Java
src/gatling/java/org/galaxio/performance/{Performance.java,cases,feeders,scenarios,*Simulation.java}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}

# Gradle + Kotlin
src/gatling/kotlin/org/galaxio/performance/{Performance.kt,cases,feeders,scenarios,*Simulation.kt}
src/gatling/resources/{simulation.conf,gatling.conf,logback.xml}
```

Keep boundaries strict:

- `cases`: request/action only. No workload. No scenario.
- `feeders`: data only. No requests.
- `scenarios`: business flow only. No injection profile.
- `simulations`: injection, protocols, max duration. No request definitions.
- `performance.scala`, `Performance.java`, or `Performance.kt`: shared protocols only.

Do not use Gradle's `src/test/*` for Gatling simulations unless the project already
customizes the `gatling` source set. Do not use Maven's `src/gatling/*` unless the
project explicitly customizes plugin/source directories.

## Imports

Scala DSL:

Base:

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

Protocol imports, only when needed:

```scala
import io.gatling.jms.Predef._
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.apache.kafka.clients.producer.ProducerConfig
import org.galaxio.gatling.amqp.Predef._
import org.galaxio.gatling.jdbc.Predef._
import org.galaxio.gatling.kafka.Predef._

import scala.concurrent.duration.DurationInt
```

Only add assertion imports when user explicitly asks for NFR/assertions.

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

Use Picatinny `SimulationConfig`.
For Java/Kotlin projects without Picatinny bindings, centralize the same config
keys in `Performance.java`/`Performance.kt` or a dedicated `Config` helper using
system properties and resource config. Keep names compatible with Scala projects.

Default params:

```scala
baseUrl
baseAuthUrl
wsBaseUrl
intensity
stagesNumber
rampDuration
stageDuration
testDuration
```

Custom params:

```scala
val kafkaUrl = getStringParam("kafkaUrl")
val pacing   = getDurationParam("pacing")
val debug    = getBooleanParam("debug", false)
```

Do not hardcode env data in Scala. Put host, login, password, topic, queue, DB URL in `simulation.conf` or pass via `-Dparam=value`.

## Cases

Case = one atomic action.

Scala HTTP:

```scala
object HttpCases {
  val getMainPage = http("GET /")
    .get("/")
    .check(status.is(200))
}
```

Java HTTP:

```java
public final class HttpCases {
  public static final ChainBuilder getMainPage =
      exec(http("GET /").get("/").check(status().is(200)));

  private HttpCases() {}
}
```

Kotlin HTTP:

```kotlin
object HttpCases {
    val getMainPage: ChainBuilder = exec(
        http("GET /").get("/").check(status().shouldBe(200))
    )
}
```

Kafka:

```scala
object KafkaCases {
  val sendMessage = kafka("send message")
    .topic("#{topic}")
    .send[String, String]("#{key}", "#{payload}")
}
```

JDBC:

```scala
object JdbcCases {
  val selectAccount = jdbc("SELECT account")
    .queryP("SELECT * FROM accounts WHERE id = {id}")
    .params("id" -> "#{accountId}")
    .check(simpleCheck(_.nonEmpty))
}
```

AMQP:

```scala
object AmqpCases {
  val publishMessage = amqp("publish message").publish
    .queueExchange("#{queue}")
    .textMessage("#{payload}")
    .messageId("#{messageId}")
}
```

JMS:

```scala
object JmsCases {
  val sendMessage = jms("send message")
    .send
    .queue("#{queue}")
    .textMessage("#{payload}")
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

CSV feeder:

```scala
val accounts = csv("accounts.csv").circular
```

Use `queue` only when each row must be unique and data volume is enough.

Do not use tiny `queue` feeder under load. Feeder ends, test fails.

## Scenarios

Scala pattern:

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

Closed model with pacing:

```scala
object ClosedPacingScenario {
  def apply(): ScenarioBuilder = new ClosedPacingScenario().scn
}

class ClosedPacingScenario {
  val scn: ScenarioBuilder = scenario("Closed Pacing Scenario")
    .forever(
      pace(pacing)
        .feed(Feeders.messageId)
        .exec(HttpCases.getMainPage),
    )
}
```

`pace` belongs inside scenario loop. Injection controls users. `pace` controls iteration rhythm.

Java pattern:

```java
public final class MainScenario {
  public static ScenarioBuilder create() {
    return scenario("Main Scenario")
        .feed(Feeders.messageId)
        .exec(HttpCases.getMainPage);
  }

  private MainScenario() {}
}
```

Kotlin pattern:

```kotlin
object MainScenario {
    fun create(): ScenarioBuilder = scenario("Main Scenario")
        .feed(Feeders.messageId)
        .exec(HttpCases.getMainPage)
}
```

## Protocols

Keep in `performance.scala`, `Performance.java`, or `Performance.kt`.

HTTP:

```scala
package object performance {
  val httpProtocol = http
    .baseUrl(baseUrl)
    .acceptHeader("application/json")
    .contentTypeHeader("application/json")
    .disableFollowRedirect
}
```

Java HTTP:

```java
public final class Performance {
  public static final HttpProtocolBuilder httpProtocol = http
      .baseUrl(System.getProperty("baseUrl"))
      .acceptHeader("application/json")
      .contentTypeHeader("application/json")
      .disableFollowRedirect();

  private Performance() {}
}
```

Kotlin HTTP:

```kotlin
object Performance {
    val httpProtocol: HttpProtocolBuilder = http
        .baseUrl(System.getProperty("baseUrl"))
        .acceptHeader("application/json")
        .contentTypeHeader("application/json")
        .disableFollowRedirect()
}
```

JDBC:

```scala
val jdbcProtocol = DB
  .url(getStringParam("dbUrl"))
  .username(getStringParam("dbUser"))
  .password(getStringParam("dbPassword"))
  .connectionTimeout(2.minutes)
```

Kafka:

```scala
val kafkaProtocol = kafka
  .producerSettings(
    ProducerConfig.ACKS_CONFIG              -> "1",
    ProducerConfig.BOOTSTRAP_SERVERS_CONFIG -> getStringParam("kafkaUrl"),
  )
  .consumeSettings(
    ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> getStringParam("kafkaUrl"),
  )
  .timeout(10.seconds)
```

AMQP:

```scala
val amqpProtocol = amqp
  .connectionFactory(
    rabbitmq
      .host(getStringParam("amqpHost"))
      .port(getIntParam("amqpPort"))
      .username(getStringParam("amqpLogin"))
      .password(getStringParam("amqpPassword"))
      .vhost("/"),
  )
  .replyTimeout(60000)
  .consumerThreadsCount(8)
  .usePersistentDeliveryMode
```

JMS:

```scala
val jmsProtocol = jms
  .connectionFactoryName("ConnectionFactory")
  .url(getStringParam("jmsUrl"))
  .credentials(getStringParam("jmsUser"), getStringParam("jmsPassword"))
```

## Simulations

Simulation = load profile only.

Scala open model, stable load:

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

Open model, stages:

```scala
class MaxPerformanceSimulation extends Simulation {
  setUp(
    MainScenario().inject(
      incrementUsersPerSec(intensity / stagesNumber)
        .times(stagesNumber)
        .eachLevelLasting(stageDuration)
        .separatedByRampsLasting(rampDuration)
        .startingFrom(0.0),
    ),
  ).protocols(httpProtocol)
    .maxDuration(testDuration)
}
```

Closed model with pacing:

```scala
class ClosedPacingSimulation extends Simulation {
  setUp(
    ClosedPacingScenario().inject(
      rampConcurrentUsers(0).to(intensity.toInt).during(rampDuration),
      constantConcurrentUsers(intensity.toInt).during(stageDuration),
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

Run:

```bash
sbt 'Gatling/testOnly org.galaxio.performance.DebugSimulation'
sbt 'Gatling/testOnly org.galaxio.performance.StabilitySimulation'
```

Java simulation shape:

```java
public class DebugSimulation extends Simulation {
  {
    setUp(
        MainScenario.create().injectOpen(atOnceUsers(1))
    ).protocols(Performance.httpProtocol);
  }
}
```

Kotlin simulation shape:

```kotlin
class DebugSimulation : Simulation() {
    init {
        setUp(
            MainScenario.create().injectOpen(atOnceUsers(1))
        ).protocols(Performance.httpProtocol)
    }
}
```

## Checks

Check technical and business success.

Good:

```scala
.check(
  status.is(200),
  jsonPath("$.id").saveAs("id"),
)
```

Do not check HTTP `200` only when response body carries business error.

Saved value exists only after successful check.

Use `checkIf` for optional branches.

## Session

Session immutable. Return changed session.

Bad:

```scala
exec { session =>
  session.set("id", "1")
  session
}
```

Good:

```scala
exec { session =>
  session.set("id", "1")
}
```

EL strings work inside Gatling DSL. Plain Scala functions need explicit session read.

Bad:

```scala
myFunction("#{id}")
```

Good:

```scala
exec { session =>
  myFunction(session("id").as[String])
  session
}
```

## Assertions And NFR

Do not add NFR/assertions by default.

Add only when user explicitly asks for NFR, SLA, assertions, or pass/fail gates.

When asked and Picatinny exists:

```scala
import org.galaxio.gatling.assertions.AssertionsBuilder.assertionFromYaml

.assertions(assertionFromYaml("src/test/resources/nfr.yml"))
```

Without Picatinny:

```scala
.assertions(
  global.successfulRequests.percent.gt(99),
  global.responseTime.percentile3.lt(1000),
)
```

## Operations

Use `maxDuration` as safety fuse.

Use groups for business transaction timings.

Use `before` and `after` only for setup/teardown outside virtual-user flow.

No `println` under load. Debug only in smoke simulation.

## Build Files

Typical `.scalafmt.conf`:

```hocon
runner.dialect = "scala213"
version = 3.10.6
binPack.parentConstructors = true
maxColumn = 128
includeCurlyBraceInSelectChains = false
align.preset = most
trailingCommas = always
```

Typical `build.sbt` shape:

```scala
enablePlugins(GatlingPlugin)

ThisBuild / scalaVersion := "2.13.18"

libraryDependencies ++= Seq(
  "io.gatling.highcharts" % "gatling-charts-highcharts" % "3.11.5" % Test,
  "io.gatling"            % "gatling-test-framework"    % "3.11.5" % Test,
  "org.galaxio"          %% "gatling-picatinny"         % "<version>",
)
```

Add protocol plugins only when used:

```scala
libraryDependencies += "io.gatling"    % "gatling-jms"           % "3.11.5" % Test
libraryDependencies += "org.galaxio" %% "gatling-jdbc-plugin"  % "<version>" % Test
libraryDependencies += "org.galaxio" %% "gatling-kafka-plugin" % "<version>" % Test
libraryDependencies += "org.galaxio" %% "gatling-amqp-plugin"  % "<version>" % Test
```

sbt plugin:

```scala
// project/plugins.sbt
addSbtPlugin("io.gatling" % "gatling-sbt" % "<gatling-sbt-plugin-version>")
```

Maven Java shape:

```xml
<dependencies>
  <dependency>
    <groupId>io.gatling.highcharts</groupId>
    <artifactId>gatling-charts-highcharts</artifactId>
    <version>${gatling.version}</version>
    <scope>test</scope>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>io.gatling</groupId>
      <artifactId>gatling-maven-plugin</artifactId>
      <version>${gatling-maven-plugin.version}</version>
    </plugin>
  </plugins>
</build>
```

Maven Scala additions:

```xml
<testSourceDirectory>src/test/scala</testSourceDirectory>
<plugin>
  <groupId>net.alchim31.maven</groupId>
  <artifactId>scala-maven-plugin</artifactId>
  <version>${scala-maven-plugin.version}</version>
  <executions>
    <execution>
      <goals>
        <goal>testCompile</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Maven Kotlin additions:

```xml
<testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
<dependency>
  <groupId>org.jetbrains.kotlin</groupId>
  <artifactId>kotlin-stdlib</artifactId>
  <version>${kotlin.version}</version>
</dependency>
<plugin>
  <groupId>org.jetbrains.kotlin</groupId>
  <artifactId>kotlin-maven-plugin</artifactId>
  <version>${kotlin.version}</version>
</plugin>
```

Gradle Scala shape:

```groovy
plugins {
  id 'scala'
  id 'io.gatling.gradle' version '<gatling-gradle-plugin-version>'
}

repositories {
  mavenCentral()
}

tasks.withType(ScalaCompile) {
  scalaCompileOptions.forkOptions.jvmArgs = ['-Xss100m']
}
```

Gradle Java shape:

```groovy
plugins {
  id 'java'
  id 'io.gatling.gradle' version '<gatling-gradle-plugin-version>'
}

repositories {
  mavenCentral()
}
```

Gradle Kotlin shape:

```kotlin
plugins {
    kotlin("jvm") version "<kotlin-version>"
    kotlin("plugin.allopen") version "<kotlin-version>"
    id("io.gatling.gradle") version "<gatling-gradle-plugin-version>"
}

repositories {
    mavenCentral()
}
```

For Gradle-only dependencies used by simulations, add them to `gatling`,
`gatlingImplementation`, or `gatlingRuntimeOnly`, not only to `implementation`.

## Do Not

Do not mix cases, scenario, simulation in one file.

Do not put injection in scenarios.

Do not put request definitions in simulations.

Do not hardcode secrets.

Do not use `Thread.sleep`; use `pause` or `pace`.

Do not use throttling as main workload model. Use injection first.

Do not use `status.is(200)` as only validation for business APIs.

Do not create clients/connections per request.

Do not mutate shared vars from virtual users unless thread-safe.

Do not ignore feeder exhaustion.

Do not add NFR gates unless user asks.

Do not use Scala 3 for Gatling plugin projects unless repo already supports it.

Do not put Gatling Gradle simulations in `src/test/*` unless the project has
explicitly customized the `gatling` source set.

Do not omit `scala-maven-plugin` in Maven Scala projects; Gatling Maven plugin v4+
does not compile Scala simulations by itself.
