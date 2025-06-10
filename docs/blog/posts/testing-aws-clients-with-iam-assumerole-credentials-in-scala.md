---
authors:
  - copperlight
categories:
  - AWS
  - IAM
  - Scala
date: 2020-01-06
draft: false
pin: false
---

# Testing AWS Clients with IAM AssumeRole Credentials in Scala

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

This technique is meant to be used with IntelliJ Scala worksheets or similar scratch code, so you
can test clients and validate their behavior. Don't check-in secrets – use on-instance credentials
with the `DefaultAWSCredentialsProviderChain` in production code.

```scala
import com.amazonaws.auth.AWSStaticCredentialsProvider
import com.amazonaws.auth.BasicSessionCredentials
import com.amazonaws.regions.Regions
import com.amazonaws.services.autoscaling.AmazonAutoScalingClientBuilder
import com.amazonaws.services.securitytoken.AWSSecurityTokenServiceClientBuilder
import com.amazonaws.services.securitytoken.model.AssumeRoleRequest

// config values

val accessKeyId = ""
val secretAccessKey = ""
val token = ""

val role: Option[String] = None
val accountId = ""
val region = Regions.US_WEST_1

// client configuration

val staticProvider = {
  role.fold {
    val basic = new BasicSessionCredentials(accessKeyId, secretAccessKey, token)
    new AWSStaticCredentialsProvider(basic)
  } { role =>
    val instanceProvider = {
      val basic = new BasicSessionCredentials(accessKeyId, secretAccessKey, token)
      new AWSStaticCredentialsProvider(basic)
    }

    val stsClient = AWSSecurityTokenServiceClientBuilder
      .standard()
      .withCredentials(instanceProvider)
      .withRegion(region)
      .build()

    val req = new AssumeRoleRequest()
      .withRoleSessionName(s"$role-testing")
      .withRoleArn(s"arn:aws:iam::$accountId:role/$role")

    val assumedCreds = stsClient.assumeRole(req).getCredentials

    val basic = new BasicSessionCredentials(
      assumedCreds.getAccessKeyId,
      assumedCreds.getSecretAccessKey,
      assumedCreds.getSessionToken
    )

    new AWSStaticCredentialsProvider(basic)
  }
}

val client = AmazonAutoScalingClientBuilder
  .standard()
  .withCredentials(staticProvider)
  .withRegion(region)
  .build()
```
