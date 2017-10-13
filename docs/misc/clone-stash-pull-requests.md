# Clone Stash Pull Requests

<small>2015-11-18</small>

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
