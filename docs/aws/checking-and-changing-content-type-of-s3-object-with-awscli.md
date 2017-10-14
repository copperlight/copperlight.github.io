# Checking and Changing Content-Type of S3 Object with AWSCLI

<div class="meta">
  <span class="date"><small>2017-03-04</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

```
aws s3api get-object \
    --bucket my.bucket \
    --key foo/bar/2017-01-26/usage.json \
    usage.json

aws s3api copy-object \
    --bucket archive \
    --content-type "application/rss+xml" \
    --copy-source archive/test/test.html \
    --key test/test.html \
    --metadata-directive "REPLACE"
```
