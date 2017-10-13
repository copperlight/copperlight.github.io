# Switching JDKs in OSX

Add the following to your `$HOME/.bash_profile`:

```bash
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
