---
authors:
  - copperlight
categories:
  - Git
  - Stash
date: 2015-11-18
draft: false
pin: false
---

# Clone Stash Pull Requests

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

<https://answers.atlassian.com/questions/179848/local-checkout-of-a-pull-request-in-stash>

```ini
[alias]
  prstash = "!f() { git fetch $1 refs/pull-requests/$2/from:$3; } ; f"
```

```bash
# where 3 is the 3rd pull request
git prstash origin 3 dest-branch
git checkout dest-branch

# to reclone
git checkout master
git prstash origin 3 dest-branch
git checkout dest-branch
```
