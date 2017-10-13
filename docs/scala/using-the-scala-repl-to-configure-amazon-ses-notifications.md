# Using the Scala REPL to Configure Amazon SES Notifications

<small>2014-06-30</small>

Let's say you want to configure bounce notifications for SES emails using the Scala REPL.  How do
you go about accessing the AWS API to make it happen?  Make sure to have the
[AWS SDK for Java API Reference](http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/) available
for additional details.

Install Scala and SBT:

```bash
brew install scala sbt
```

Create an AWS credentials file at `~/.aws/credentials` that is formatted like so:

```ini
[default]
aws_access_key_id=...
aws_secret_access_key=...
aws_session_token=...
```

Create a `build.sbt` file that looks like:

```scala
name := "aws-sdk-client"

version := "1.0"

scalaVersion := "2.11.0"

libraryDependencies ++= Seq(
    "commons-logging" % "commons-logging" % "1.1.3",
    "com.amazonaws" % "aws-java-sdk" % "1.8.2"
)
```

You can then engage the Scala REPL to talk to the AWS API like so (Scala will download the AWS SDK
from Maven Central automatically):

```bash
sbt
console
```

```scala
import com.amazonaws.auth._
import com.amazonaws.auth.profile._
import com.amazonaws.services.simpleemail._
import com.amazonaws.services.simpleemail.model._
import com.amazonaws.services.sns._
import com.amazonaws.services.sns.model._

val c = new AmazonSNSClient(
    new AWSCredentialsProviderChain(
        // attempt on-instance credentials first
        new InstanceProfileCredentialsProvider(),
        // fallback to aws credentials file
        new ProfileCredentialsProvider()
    )
)

c.setEndpoint("sns.us-east-1.amazonaws.com")

c.createTopic("ses-email-bounce")

res0: com.amazonaws.services.sns.model.CreateTopicResult =
{TopicArn: arn:aws:sns:us-east-1:000000000000:ses-email-bounce}

val c = new AmazonSimpleEmailServiceClient(
    new AWSCredentialsProviderChain(
        // attempt on-instance credentials first
        new InstanceProfileCredentialsProvider(),
        // fallback to aws credentials file
        new ProfileCredentialsProvider()
    )
)

c.setEndpoint("email.us-east-1.amazonaws.com")

c.getSendQuota()

res1: com.amazonaws.services.simpleemail.model.GetSendQuotaResult =
{Max24HourSend: 100.0,MaxSendRate: 10.0,SentLast24Hours: 1.0}

c.getSendStatistics()

res2: com.amazonaws.services.simpleemail.model.GetSendStatisticsResult =
{SendDataPoints: [{Timestamp: ...,DeliveryAttempts: 0,Bounces: 0,Complaints: 0,Rejects: 0}, ...

c.getIdentityVerificationAttributes(
    new GetIdentityVerificationAttributesRequest()
        .withIdentities("some.email@example.com")
)

res3: com.amazonaws.services.simpleemail.model.GetIdentityVerificationAttributesResult =
{VerificationAttributes: {some.email@mydomain.com={VerificationStatus: Success,}}}

c.setIdentityNotificationTopic(
    new SetIdentityNotificationTopicRequest()
        .withIdentity("some.email@example.com")
        .withNotificationType("Bounce")
        .withSnsTopic(s"arn:aws:sns:us-east-1:$accountId:ses-email-bounce")
)

c.getIdentityNotificationAttributes(
    new GetIdentityNotificationAttributesRequest()
        .withIdentities("some.email@example.com")
)
```

One of the nice things about using the Scala REPL is that you get some useful tab completions (SES
object here):

```
scala> c.
addRequestHandler                   sendEmail
asInstanceOf                        sendRawEmail
deleteIdentity                      setConfiguration
deleteVerifiedEmailAddress          setEndpoint
getCachedResponseMetadata           setIdentityDkimEnabled
getIdentityDkimAttributes           setIdentityFeedbackForwardingEnabled
getIdentityNotificationAttributes   setIdentityNotificationTopic
getIdentityVerificationAttributes   setRegion
getRequestMetricsCollector          setServiceNameIntern
getSendQuota                        setSignerRegionOverride
getSendStatistics                   setTimeOffset
getServiceName                      shutdown
getSignerByURI                      toString
getSignerRegionOverride             verifyDomainDkim
getTimeOffset                       verifyDomainIdentity
isInstanceOf                        verifyEmailAddress
listIdentities                      verifyEmailIdentity
listVerifiedEmailAddresses          withTimeOffset
removeRequestHandler
```

Once you have a feel for how the Scala REPL works with a library, you can create alternate entry
point for sbt called `scalas` which allows you to write full Scala scripts with library
dependencies.  See [Scripts, REPL, and Dependencies](http://www.scala-sbt.org/release/docs/Scripts.html)
for additional details.  In brief, create a `scalas` script and make it available on your `PATH`:

```bash
#!/bin/sh
test -f ~/.sbtconfig && . ~/.sbtconfig
exec java \
    -Xms512M \
    -Xmx1536M \
    -Xss1M \
    -XX:+CMSClassUnloadingEnabled \
    -XX:MaxPermSize=256M \
    ${SBT_OPTS} \
    -jar /usr/local/Cellar/sbt/0.13.2/libexec/sbt-launch.jar \
    -Dsbt.main.class=sbt.ScriptMain \
    "$@"
```

You can then write Scala scripts like so:

```scala
#!/usr/bin/env scalas

/***
    scalaVersion := "2.11.0"

    libraryDependencies ++= Seq(
        "commons-logging" % "commons-logging" % "1.1.3",
        "com.amazonaws" % "aws-java-sdk" % "1.8.2"
    )
*/

import com.amazonaws.auth._
import com.amazonaws.auth.profile._
import com.amazonaws.services.simpleemail._
import com.amazonaws.services.simpleemail.model._

val c = new AmazonSimpleEmailServiceClient(
    new AWSCredentialsProviderChain(
        // attempt on-instance credentials first
        new InstanceProfileCredentialsProvider(),
        // fallback to aws credentials file
        new ProfileCredentialsProvider()
    )
)

c.setEndpoint("email.us-east-1.amazonaws.com")

println(c.getSendQuota())

println(c.getSendStatistics().getSendDataPoints.get(0))

println(c.getSendStatistics().getSendDataPoints.size)
```

Execute the script as follows:

```
$ ./demo.scala
.
.
.
{Timestamp: Sat Jul 05 04:01:00 PDT 2014,DeliveryAttempts: 3,Bounces: 0,Complaints: 0,Rejects: 0}
1301
```
