---
authors:
  - copperlight
categories:
  - Gradle
  - Scala
date: 2015-07-04
draft: false
pin: false
---

# Starting Scala REPLs with Gradle and SBT

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

These examples are tailored for usage with the Atomic Scala exercises, but they demonstrate the
principles.

**build.gradle**
```groovy
apply plugin: 'scala'

repositories {
    mavenCentral()
}

ext {
    versions = [
        commons_math3: '3.5',
        jline: '2.12.1',
        scala: '2.11.6'
    ]
}

dependencies {
    compile "org.apache.commons:commons-math3:${versions.commons_math3}"
    compile "org.scala-lang:scala-library:${versions.scala}"

    runtime "jline:jline:${versions.jline}"
    runtime "org.scala-lang:scala-compiler:${versions.scala}"
}

sourceSets {
    main {
        scala {
            srcDirs = ['.']
            include 'AtomicTest.scala'
        }
    }
}

scalaConsole.dependsOn(build)
scalaConsole.classpath += sourceSets.main.runtimeClasspath

// usage: gradlew scalaConsole -q
```

**build.sbt**
```scala
scalaVersion := "2.11.6"

sources in Compile <<= (sources in Compile).map(_ filter(_.name == "AtomicTest.scala"))

libraryDependencies += "org.apache.commons" % "commons-math3" % "3.5"

initialCommands in console := """
    |import org.apache.commons.math3._
    |import com.atomicscala.AtomicTest._
    |""".stripMargin

// usage: sbt console
```
