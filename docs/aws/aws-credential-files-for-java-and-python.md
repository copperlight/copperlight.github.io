# AWS Credential Files for Java and Python

<div class="meta">
  <span class="date"><small>2014-07-14</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

Each of the AWS tools has slightly different expectations about the location and naming of the
credentials file and the various properties within it.  It seems like the Python tools are moving
closer to the Java standard as they iterate through releases, but it is still necessary to use a
patchwork solution to be able to have a unified credentials file.

## Java SDK

<table>
    <tr> <th>Version: <td>"com.amazonaws" % "aws-java-sdk" % "1.8.2"
    <tr> <th>Installation: <td>sbt
    <tr> <th>Link: <td><a href="http://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/profile/ProfilesConfigFile.html">AWS Java SDK Class ProfilesConfigFile</a>
</table>

The standard location for the credentials file is `~/.aws/credentials`, which can be overridden with
the `AWS_CREDENTIAL_PROFILES_FILE` environment variable or by specifying an alternate file location
in the constructor.  The format of this file is described below:

```
[default]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
aws_session_token=testSessionToken

[test-user]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
aws_session_token=testSessionToken
```

## Java Command Line Tools

<table>
    <tr> <th>Version: <td>1.6.13.0
    <tr> <th>Installation: <td>brew install ec2-api-tools
    <tr> <th>Link: <td><a href="http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/set-up-ec2-cli-linux.html">Setting Up the Amazon EC2 Command Line Interface Tools on Linux/Unix and Mac OS X</a></td>
</table>

The standard configuration is to use environment variables, since these tools have not been updated
to read the standard AWS credentials file.  Add the following to your `~/.bash_profile`, to link the
required data to the standard credentials file and allow for session tokens:

```
export AWS_CREDENTIAL_FILE="$HOME/.aws/credentials"
export AWS_ACCESS_KEY="$(grep aws_access_key_id $AWS_CREDENTIAL_FILE |cut -d= -f2)"
export AWS_SECRET_KEY="$(grep aws_secret_access_key $AWS_CREDENTIAL_FILE |cut -d= -f2)"
export AWS_DELEGATION_TOKEN="$(grep aws_session_token $AWS_CREDENTIAL_FILE |cut -d= -f2)"
```

## Python Boto

<table>
    <tr> <th>Version: <td>boto==2.31.1 <br> botocore==0.56.0
    <tr> <th>Installation: <td>pip install boto
    <tr> <th>Link: <td><a href="http://boto.readthedocs.org/en/latest/boto_config_tut.html">Boto Config</a></td>
</table>

The latest version of boto needs to have `aws_security_token` defined, rather than
`aws_session_token`, in the credentials file.  The simplest solution for this is to duplicate the
token between both names; the Java SDK will throw the following log message when reading the extra
property, but will work as expected: `INFO: Skip unsupported property name aws_security_token in
profile [default].`  Boto will not throw log messages about the existence of the `aws_session_token`
property.

## AWS CLI

<table>
    <tr> <th>Version: <td>aws-cli/1.3.22
    <tr> <th>Installation: <td>pip install awscli
    <tr> <th>Link: <td><a href="http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html">Configuring the AWS Command Line Interface</a></td>
</table>

The standard location for the credentials file is `~/.aws/config`, which can be overridden with the
`AWS_CREDENTIAL_FILE` environment variable.  The latest version of this tool accepts the Java SDK
credential file format as-is, including the use of `aws_session_token`, whereas previous versions
wanted `aws_security_token` instead.  When you have multiple profiles in the credentials file, you
can select a profile with the tool like so:

```
aws --profile test-user s3 ls
```

## Unified Solution

The best approach for creating a unified credentials file is to follow the Java credentials file
format as closely as possible, while redirecting the Python tools to that file and adding properties
to cover the corner cases.

To do this, create a `~/.aws/credentials` file that duplicates the necessary properties:

```
[default]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
aws_session_token=testSessionToken
aws_security_token=testSessionToken

[test-user]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
aws_session_token=testSessionToken
aws_security_token=testSessionToken

[prod-user]
aws_access_key_id=testAccessKey
aws_secret_access_key=testSecretKey
aws_session_token=testSessionToken
aws_security_token=testSessionToken
```

And add a section to your `~/.bash_profile`:

```
export AWS_CREDENTIAL_FILE="$HOME/.aws/credentials"
export AWS_ACCESS_KEY="$(grep aws_access_key_id $AWS_CREDENTIAL_FILE |cut -d= -f2)"
export AWS_SECRET_KEY="$(grep aws_secret_access_key $AWS_CREDENTIAL_FILE |cut -d= -f2)"
export AWS_DELEGATION_TOKEN="$(grep aws_session_token $AWS_CREDENTIAL_FILE |cut -d= -f2)"
```

With this configuration, you should be able to move seamlessly between the various Java and Python
tools available for AWS.
