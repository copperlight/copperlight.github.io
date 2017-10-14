# Using Gradle to Run Scala Projects Locally

<div class="meta">
  <span class="date"><small>2017-10-12</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

Running projects locally in a minimal fashion is often useful for understanding the code, using a
debugger and performing interactive integration testing.

You can run REPLs with `./gradlew`, but due to the input fiddling that is going on, it's very
distracting and not effective, especially if you need to build up anything more than simple state.
You will be better served by using the [IntelliJ Scala Worksheet], which makes it easy to gain
access to the libraries you have included in your build.

[IntelliJ Scala Worksheet]: https://www.jetbrains.com/help/idea/working-with-scala-worksheet.html

```groovy
buildscript {
    dependencies {
        classpath "gradle.plugin.com.github.maiflai:gradle-scalatest:0.14"
        classpath "org.akhikhl.gretty:gretty:2.0.0"
    }
}

apply plugin: 'com.github.maiflai.scalatest'
apply plugin: 'org.akhikhl.gretty'

tasks.withType(ScalaCompile) {
    scalaCompileOptions.setAdditionalParameters([
        '-deprecation',
        '-unchecked',
        '-Xexperimental',
        '-Xlint:_,-infer-any',
        '-feature',
        '-Ydelambdafy:method'
    ])
}

// for servlet applications
gretty {
    httpPort = 7101
    contextPath = '/'
    servletContainer = 'jetty7'
    jvmArgs = [
        "-Dlog4j.configurationFile=" + System.getProperty("user.dir") + "/src/main/resources/log4j_dev.xml"
    ]
}

// for main class applications
task runService(dependsOn: 'classes', type: JavaExec) {
    main = 'com.example.app.Main'
    classpath sourceSets.main.runtimeClasspath
}
```

```
./gradlew appRun
./gradlew runService
```
