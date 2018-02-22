<div class="meta">
  <span class="date"><small>2018-02-22</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

JMH is the Java Microbenchmark Harness provided by the OpenJDK project.  It can
be a useful tool for validating assumptions about the performance of small code
sections subject to many iterations.

See the following links for more details:

* [OpenJDK Code Tools: jmh](http://openjdk.java.net/projects/code-tools/jmh/)
* [JMH Tutorial](http://tutorials.jenkov.com/java-performance/jmh.html)

## Gradle

A [Gradle plugin](https://github.com/melix/jmh-gradle-plugin) is available for JMH.

Project layout:

```
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    └── jmh
        └── scala
            └── io
                └── github
                    └── brharrington
                        ├── AtomicLongUpdate.scala
                        ├── CurrentTimeMillis.scala
                        ├── NanoTime.scala
                        └── TimeLimiting.scala
```

The `build.gradle` configuration:

```
plugins {
    id 'scala'
    id 'idea'
    id 'me.champeau.gradle.jmh' version '0.4.5'
}

repositories {
    jcenter()
}

dependencies {
    compile 'com.netflix.servo:servo-core:0.12.17'
    compile 'org.scala-lang:scala-library:2.12.4'
}

idea {
    module {
        testSourceDirs += sourceSets.jmh.scala.srcDirs
    }
}
```

Run the tests:

```
./gradlew jmh
```

## sbt

An [sbt plugin](https://github.com/ktoso/sbt-jmh) is available for JMH.

Example [project layout](https://github.com/brharrington/misc-jmh):

```
.
├── build.sbt
├── project
│   ├── build.properties
│   ├── plugins.sbt
│   ├── sbt
│   └── sbt-launch-1.0.0.jar
└── src
    └── main
        └── scala
            └── io
                └── github
                    └── brharrington
                        ├── AtomicLongUpdate.scala
                        ├── CurrentTimeMillis.scala
                        ├── NanoTime.scala
                        └── TimeLimiting.scala
```

The `build.sbt` configuration:

```
name := "misc-jmh"

organization := "io.github.brharrington"

scalaVersion := "2.12.4"

enablePlugins(JmhPlugin)

libraryDependencies ++= Seq(
    "com.netflix.servo" % "servo-core" % "0.12.17"
)
```

The `plugins.sbt` configuration:

```
addSbtPlugin("pl.project13.scala" % "sbt-jmh" % "0.2.27")
```

Run the tests:

```
./project/sbt jmh:run
```

## Example Test Classes

```
package io.github.brharrington

import java.util.concurrent.atomic.AtomicLong

import org.openjdk.jmh.annotations.Benchmark
import org.openjdk.jmh.annotations.Scope
import org.openjdk.jmh.annotations.State
import org.openjdk.jmh.infra.Blackhole

@State(Scope.Thread)
class AtomicLongUpdate {

  private val value = new AtomicLong()

  @Benchmark
  def incrementAndGet(bh: Blackhole): Unit = {
    bh.consume(value.incrementAndGet())
  }

  @Benchmark
  def addAndGet(bh: Blackhole): Unit = {
    bh.consume(value.addAndGet(42))
  }

  @Benchmark
  def get(bh: Blackhole): Unit = {
    bh.consume(value.get())
  }

}
```
