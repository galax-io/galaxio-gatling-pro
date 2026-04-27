---
name: galaxio-gatling-pro
description: >
  Use when creating, reviewing, or refactoring Gatling Scala performance tests
  in Galaxio style: sbt projects, Gatling 3.11.x, Scala 2.13, Picatinny
  config/feeders, cases/feeders/scenarios/simulations layout,
  HTTP/JDBC/JMS/Kafka/AMQP protocols, open and closed workload models,
  smoke/debug simulations, and scalafmt-ready code.
---

# Galaxio Gatling Pro

## Core Rules

Use Gatling `3.11.x`, Scala `2.13.x`, sbt, Scala DSL.

Prefer `org.galaxio.gatling-picatinny` helpers when repo has dependency.

Generated code must pass:

```bash
sbt scalafmtAll scalafmtSbt
```

When changing existing repo, follow local style first. If no style, use this skill.

## Project Layout

Expected layout:

```text
performance/
  cases/       # atomic actions: HTTP, Kafka, JDBC, AMQP, JMS
  feeders/     # custom feeders; prefer Picatinny feeders
  scenarios/   # flows built from cases
src/test/scala/org/galaxio/performance/
  performance.scala # package object with protocols
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
- `performance.scala`: shared protocols only.

## Imports

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

## Config

Use Picatinny `SimulationConfig`.

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

HTTP:

```scala
object HttpCases {
  val getMainPage = http("GET /")
    .get("/")
    .check(status.is(200))
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

Pattern:

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

## Protocols

Keep in `performance.scala`.

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

Open model, stable load:

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
