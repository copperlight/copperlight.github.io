# Reset Jenkins Build Number

<small>2017-03-14</small>

<http://stackoverflow.com/questions/20901791/how-to-reset-build-number-in-jenkins>

Go to the Jenkins Script Console and enter:

```groovy
item = Jenkins.instance.getItemByFullName("your-job-name-here")

// remove all build history
item.builds.each() { build ->
  build.delete()
}

item.updateNextBuildNumber(1)
```
