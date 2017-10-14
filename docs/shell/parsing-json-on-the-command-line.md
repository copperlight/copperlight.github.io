# Parsing JSON on the Command Line

<div class="meta">
  <span class="date"><small>2014-07-16</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

With more APIs moving to JSON, being able to parse it at the command line allows you to write more
sophisticated shell scripts that can interact with your favorite services.  Most first attempts at
JSON parsing using some variation of the following to get the job done, which initially seems
reasonable:

```bash
curl -s http://endpoint.info \
    |python -mjson.tool \
    |grep foo \
    |cut -d: -f2 \
    |sed -e 's/"//g'
```

However, this can rapidly get out of hand, if you have key duplication, complex nested structures or
you need to pull in all of the elements of a list.  For awhile,
[underscore-cli](https://github.com/ddopson/underscore-cli) was my favorite fully-featured JSON
parser, but I found its documentation somewhat lacking and it hasn't seen serious development since
November 2012.  Since then, I found [jq](http://stedolan.github.io/jq/) which has a beautiful,
well-written [manual](http://stedolan.github.io/jq/manual/) with many usage examples and it is under
active development.  It also has the benefit of being written in C, which helps speed and it has a
fairly concise descriptor language.  To install:

```bash
brew install jq
```

Then you can do things like flattening a complex JSON structure into a simple CSV:

```bash
PAYLOADS=$( curl -s $URL |jq '.payloads' )
if [ "$PAYLOADS" != "[]" ]; then
    echo $PAYLOADS \
        | jq -r '.[] | .minutes[].payload.items[] | [.actions.actionTime, (.actions | {actions} | .actions.email.state), .sourceInstance, (.actions | {actions} | .actions.email.info )] | @csv'
fi
```

As a bonus feature, if you have to deal in XML rather than JSON, then
[xmlstarlet](http://xmlstar.sourceforge.net/) is a good choice for handling it.  Naturally,
installation:

```bash
brew install xmlstarlet
```

Once you have xmlstarlet, you can do things like pull the build number out of an Atlassian Bamboo
HTML page, which was necessary when they did not have an API call available to report the version
number of the last known good build for a project:

```bash
curl -s --insecure https://bamboo.local/browse/${BUILDKEY}${BUILD} \
    |tidy -asxhtml -numeric --force-output true 2>/dev/null \
    |xmlstarlet sel -N x="http://www.w3.org/1999/xhtml" -t -m "//x:div[@id='sr-build']/x:h2/x:a" -v "." \
    |sed -e "s/#//g"
```
