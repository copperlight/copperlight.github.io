---
authors:
  - copperlight
categories:
  - Jenkins
date: 2017-03-14
draft: false
pin: false
---

# Reset Jenkins Build Number

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

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
