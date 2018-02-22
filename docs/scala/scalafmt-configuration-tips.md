# Scalafmt Configuration Tips

<div class="meta">
  <span class="date"><small>2018-02-22</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

[Scalafmt](http://scalameta.org/scalafmt/) is a code formatter for Scala.  It is available as
an [IntelliJ plugin](https://plugins.jetbrains.com/plugin/8236-scalafmt) and a CLI version can
be installed using [Homebrew](http://scalameta.org/scalafmt/#Homebrew).

## Configuration

The following `.scalafmt.conf` configuration is recommended for use with Insight projects. It
should be created in the project's root directory.

```
style = defaultWithAlign

align.openParenCallSite = false
align.openParenDefnSite = false
align.tokens = [{code = "->"}, {code = "<-"}, {code = "=>", owner = "Case"}]
continuationIndent.callSite = 2
continuationIndent.defnSite = 2
danglingParentheses = true
indentOperator = spray
maxColumn = 100
newlines.alwaysBeforeTopLevelStatements = true
project.excludeFilters = [".*\\.sbt"]
rewrite.rules = [RedundantParens, SortImports]
spaces.inImportCurlyBraces = false
unindentTopLevelOperators = true
```

The `maxColumn` value is chosen so that it is easy to do side-by-side views of code and tests,
while also avoiding wrapping in diffs on GitHub and Stash.

Alignment is allowed for case statements `=>` and map assignments `->` and it is disallowed for
assignment `=`.  This option taken together with the `openParen.*Site` configuration minimizes
the amount of whitespace fiddling that will occur in code changes, which should lead to more
readable diffs.

## Usage Tips

* Blank lines separate alignment blocks.
* For sections of code that require formatting which does not follow the scalafmt conventions,
use [format:off](http://scalameta.org/scalafmt/#//format:off) blocks to disable the formatter.
* When formatter changes include altering whitespace to align operators, it can make diffs more
difficult to read.  You can use `git diff -w` to ignore whitespace changes.
* To enable formatting on file save in IntelliJ: IntelliJ IDEA > Preferences > Tools > Scalafmt >
check: Format on file save.

## Enforce Formatting

### Gradle

A [Scalafmt Gradle plugin](https://github.com/alenkacz/gradle-scalafmt) is available, which can be
used to enforce scalafmt conventions for your project.  It expects the configuration file to be
located in the root of your project.

Add the following to your `build.gradle` file:

```
buildscript {
    dependencies {
        classpath 'cz.alenkacz:gradle-scalafmt:1.6.0'
    }
}

apply plugin: 'scalafmt'
```

This adds a task `checkScalafmtAll` to your project, which can be used to verify that your project
files adhere to the established configuration.

### Sbt

A [Scalafmt sbt plugin](https://github.com/lucidsoftware/neo-sbt-scalafmt) is available, which can
be used to enforce scalafmt conventions for your project.  It expects the configuration file to be
located in the root of your project.

```
scalafmt::test test:scalafmt::test
```
