---
authors:
  - copperlight
categories:
  - AWS
  - STS
date: 2016-05-19
draft: false
pin: false
---

# Assume Role with AWSCLI

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

If you have either static or instance profile credentials that grant you STS permissions, then you
can gather a set of time-limited role credentials as follows:

```
#!/bin/bash

TEST_CREDENTIALS=$( \
  aws sts assume-role \
  --role-arn arn:aws:iam::$AWS_ACCOUNT_ID_1:role/$ROLE_NAME \
  --role-session-name $USER \
  |jq '.Credentials'
)

PROD_CREDENTIALS=$( \
  aws sts assume-role \
  --role-arn arn:aws:iam::$AWS_ACCOUNT_ID_2:role/$ROLE_NAME \
  --role-session-name $USER \
  |jq '.Credentials'
)

cat >>$HOME/.aws/credentials <<EOF
[test-$ROLE_NAME]
aws_access_key_id=$(echo $TEST_CREDENTIALS |jq -r '.AccessKeyId')
aws_secret_access_key=$(echo $TEST_CREDENTIALS |jq -r '.SecretAccessKey')
aws_session_token=$(echo $TEST_CREDENTIALS |jq -r '.SessionToken')
expiration=$(echo $TEST_CREDENTIALS |jq -r '.Expiration')

[prod-$ROLE_NAME]
aws_access_key_id=$(echo $PROD_CREDENTIALS |jq -r '.AccessKeyId')
aws_secret_access_key=$(echo $PROD_CREDENTIALS |jq -r '.SecretAccessKey')
aws_session_token=$(echo $PROD_CREDENTIALS |jq -r '.SessionToken')
expiration=$(echo $PROD_CREDENTIALS |jq -r '.Expiration')
EOF
```
