# IntelliJ Configuration Tips

<div class="meta">
  <span class="date"><small>2017-10-05</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

This page provides a few tips on configuring IntelliJ for working with Scala code.

## Optimize Imports

The preference for organizing imports is to keep them on a single line per import, rather than
bunching them together with curly braces.  You can enforce this behavior with IntelliJ by modifying
the following configuration:

* IntelliJ IDEA > Preferences
* Code Style > Scala > Imports
* Uncheck: *Collect imports with the same prefix into one import*

When you run Code > Optimize Imports on a source file, it will unbundle imports
and leave them with a single one per line.

## Code Style

Navigate to these configuration options as follows:

* IntelliJ IDEA > Preferences
* Editor > Code Style > Scala

### Method declaration parameters

* Uncheck: Align when multiline

This prevents Intellij from indenting parameter lists when cutting and pasting code within a file.

## Method Signature Warnings

Navigate to these configuration options as follows:

* IntelliJ IDEA > Preferences
* Editor > Inspections > Scala > Method Signature

### Method with accessor-like name has Unit result type

The intent of this warning is to cover the following case:

> Methods that follow JavaBean naming contract for accessors are expected to have no side effects.
> However, methods with a result type of Unit are only executed for their side effects.
>
> Refer to Programming in Scala, 2.3 Define some functions

This indicates poor naming of the method involved, but depending on the code you are working with,
there may be multiple examples of this present.  You may want to set this to be a weak warning until
you can update method names to be more descriptive.

### Method with Unit result type is defined like procedure

This inspection is disabled by default.  It should be enabled.

> It is not recommended to use procedure-like syntax for methods with Unit return type. It is
> inconsistent, may lead to errors and will be deprecated in future versions of Scala.
>
> Reference: The talk "Scala with Style" by Martin Odersky at ScalaDays 2013
>
> Hint: You can use Analyze / Run Inspection by Name (Ctrl+Alt+Shift+I) to apply this inspection
> to the whole project

## Scala Worksheet

This is a good way to gain access to an interactive repl-style experience for
evaluating Scala code.  As long as the worksheet file is established within
your `main` directory, you will have access to import all the dependencies of
your project.

However, the default configuration of the Scala Worksheet will not be able to find
Akka actor resources ([SCL-9229](https://youtrack.jetbrains.com/issue/SCL-9229)).
You will receive the following error:

```
com.typesafe.config.ConfigException$Missing:
  No configuration setting found for key 'akka'
```

The work-around for this issue is as follows:

* IntelliJ IDEA > Preferences
* Languages & Frameworks > Scala > Worksheet
* Uncheck: *Run worksheet in the compiler process*

## Zero-Latency Typing

See [Experimental Zero-latency Typing in IntelliJ IDEA 15 EAP] for an explanation of this feature.
To implement it, follow these steps:

* Help > Edit Custom Properties...
* Add `editor.zero.latency.typing=true`
* Restart IntelliJ

At some point, this will become the default.

[Experimental Zero-latency Typing in IntelliJ IDEA 15 EAP]: https://blog.jetbrains.com/idea/2015/08/experimental-zero-latency-typing-in-intellij-idea-15-eap/
