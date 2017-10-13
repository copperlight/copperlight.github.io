# Checking and Changing Content-Type of S3 Object with AWSCLI

<small>2017-03-04</small>

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
