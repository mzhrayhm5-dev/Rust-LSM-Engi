import sbt._
import Keys._

// Core versions - update carefully, scalaz 7.3.x requires scala 2.12+
val scalazVersion = "7.3.7"
val specs2Version = "4.20.0"

ThisBuild / scalaVersion       := "2.13.12"
ThisBuild / crossScalaVersions := Seq("2.12.18", "2.13.12")
ThisBuild / organization       := "com.github.jdegoes"
ThisBuild / version            := "1.0.0-SNAPSHOT"
ThisBuild / homepage           := Some(url("https://github.com/jdegoes/blueeyes"))

val commonDeps = Seq(
  "org.scalaz"     %% "scalaz-core"     % scalazVersion,
  "org.scalaz"     %% "scalaz-effect"   % scalazVersion,
  "org.specs2"     %% "specs2-core"     % specs2Version % Test,
  "org.scalacheck" %% "scalacheck"      % "1.17.0"      % Test,
  // logback test-only, keep off compile scope
  "ch.qos.logback" %  "logback-classic" % "1.4.14"      % Test
)

val allSettings = Seq(
  libraryDependencies ++= commonDeps,
  scalacOptions ++= Seq(
    "-deprecation",
    "-unchecked",
    "-feature",
    "-encoding", "utf8",
    "-Xfatal-warnings"
  ),
  resolvers ++= Seq(
    "Sonatype Releases"  at "https://oss.sonatype.org/content/repositories/releases",
    // snapshots needed for scalaz nightlies during active dev
    "Sonatype Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots",
    "Typesafe"           at "https://repo.typesafe.com/typesafe/releases/"
  ),
  Test / publishArtifact := false,
  pomIncludeRepository   := { _ => false },
  publishMavenStyle      := true,
  pomExtra :=
    <licenses>
      <license>
        <name>MIT license</name>
        <url>https://opensource.org/licenses/MIT</url>
        <distribution>repo</distribution>
      </license>
    </licenses>
    <scm>
      <connection>scm:git:git@github.com:jdegoes/blueeyes.git</connection>
      <developerConnection>scm:git:git@github.com:jdegoes/blueeyes.git</developerConnection>
      <url>https://github.com/jdegoes/blueeyes</url>
    </scm>
    <developers>
      <developer><id>jdegoes</id><name>John De Goes</name></developer>
      <developer><id>nuttycom</id><name>Kris Nuttycombe</name></developer>
      <developer><id>mlagutko</id><name>Michael Lagutko</name></developer>
      <developer><id>dchenbecker</id><name>Derek Chen-Becker</name></developer>
    </developers>
)

// build order matters - dependencies defined before dependents
lazy val util         = project.settings(allSettings)
lazy val json         = project.settings(allSettings)
lazy val akka_testing = project.settings(allSettings).dependsOn(util)
lazy val bkka         = project.settings(allSettings).dependsOn(util)
lazy val core         = project.settings(allSettings).dependsOn(util, json, bkka, akka_testing)
lazy val test         = project.settings(allSettings).dependsOn(core)
// json test->test: shares test fixtures, not ideal but works for now
// TODO: extract shared test utilities to a dedicated module
lazy val mongo        = project.settings(allSettings).dependsOn(core, json % "test->test")

lazy val root = project.in(file("."))
  .withId("blueeyes")
  .settings(allSettings)
  .aggregate(util, json, akka_testing, bkka, core, test, mongo)

// actor module parked - blocked on akka migration, see issue #47
// lazy val actor = project.settings(allSettings)
