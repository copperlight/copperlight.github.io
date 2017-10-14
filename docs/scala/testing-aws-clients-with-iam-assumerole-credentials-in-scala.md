# Testing AWS Clients with IAM AssumeRole Credentials in Scala

<div class="meta">
  <span class="date"><small>2017-09-28</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

This technique is meant to be used with IntelliJ Scala worksheets or similar scratch code, so you
can test clients and validate their behavior. Don't check-in secrets – use on-instance credentials
with the `DefaultAWSCredentialsProviderChain` in production code.

```scala
import com.amazonaws.auth.AWSStaticCredentialsProvider
import com.amazonaws.auth.BasicSessionCredentials
import com.amazonaws.services.securitytoken.AWSSecurityTokenServiceClientBuilder
import com.amazonaws.services.securitytoken.model.AssumeRoleRequest
import com.amazonaws.ClientConfiguration
import com.amazonaws.regions.Regions
import com.amazonaws.services.sqs.AmazonSQSClientBuilder
import scala.collection.JavaConverters._

// curl http://169.254.169.254/latest/meta-data/iam/security-credentials/MyProfile
val instanceCreds = new BasicSessionCredentials(
  "AccessKeyId",
  "SecretAccessKey",
  "Token"
)

val stsClient = AWSSecurityTokenServiceClientBuilder
  .standard()
  .withCredentials(new AWSStaticCredentialsProvider(instanceCreds))
  .withRegion(Regions.US_EAST_1)
  .build()

val accountId = ""
val role = ""

val assumeRoleResult = stsClient.assumeRole(
  new AssumeRoleRequest()
    .withRoleSessionName("TestAssumeRole")
    .withRoleArn(s"arn:aws:iam::$accountId:role/$role")
)

val stsCredentials = assumeRoleResult.getCredentials

val assumeCreds = new BasicSessionCredentials(
  stsCredentials.getAccessKeyId,
  stsCredentials.getSecretAccessKey,
  stsCredentials.getSessionToken
)

val sqsClient = AmazonSQSClientBuilder
  .standard()
  .withCredentials(new AWSStaticCredentialsProvider(assumeCreds))
  .withClientConfiguration(new ClientConfiguration())
  .withRegion(Regions.US_EAST_1)
  .build()

sqsClient.listQueues().getQueueUrls.asScala.foreach(println(_))
```
