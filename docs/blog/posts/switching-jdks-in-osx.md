---
authors:
  - copperlight
categories:
  - CLI
  - Java
  - macOS
date: 2015-02-03
draft: false
pin: false
---

# Switching JDKs in OSX

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

Add the following to your `$HOME/.bash_profile`:

```shell
export JAVA7_HOME="$(/usr/libexec/java_home -v 1.7)"
export JAVA8_HOME="$(/usr/libexec/java_home -v 1.8)"
export JAVA_HOME=$JAVA8_HOME

switch_java() {
    if echo $JAVA_HOME |grep -q 1.8; then
        export JAVA_HOME=$JAVA7_HOME
    else
        export JAVA_HOME=$JAVA8_HOME
    fi
    echo "JAVA_HOME=$JAVA_HOME"
}
```
