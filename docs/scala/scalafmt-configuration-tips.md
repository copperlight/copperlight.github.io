# Scalafmt Configuration Tips

[Scalafmt](http://scalameta.org/scalafmt/) is a code formatter for Scala.  It is available as
an [IntelliJ plugin](https://plugins.jetbrains.com/plugin/8236-scalafmt) and a CLI version can
be installed using [Homebrew](http://scalameta.org/scalafmt/#Homebrew).

The following `.scalafmt.conf` configuration is recommended for use with Insight projects:

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

This config file should be created in the project's root directory or the user's home directory. To
enable formatting on file save in IntelliJ: IntelliJ IDEA > Preferences > Tools > Scalafmt > check:
Format on file save.

Blank lines separate alignment blocks.

For sections of code that require formatting which does not follow the scalafmt conventions,
use [format:off](http://scalameta.org/scalafmt/#//format:off) blocks to disable the formatter.

When formatter changes include altering whitespace to align operators, it can make diffs more
difficult to read.  You can use `git diff -w` to ignore whitespace changes.

The `maxColumn` value is chosen so that it is easy to do side-by-side views of code and tests,
while also avoiding wrapping in diffs on GitGub and Stash.

Alignment is allowed for case statements `=>` and map assignments `->` and it is disallowed for
assignment `=`.  This option taken together with the `openParen.*Site` configuration minimizes
the amount of whitespace fiddling that will occur in code changes, which should lead to more
readable diffs.
