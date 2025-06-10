---
authors:
  - copperlight
categories:
  - Python
  - Google API
date: 2016-06-03
draft: false
pin: false
---

# Accessing Google APIs with Python

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

## Links

* <https://developers.google.com/admin-sdk/directory/v1/quickstart/python#prerequisites>
* <https://developers.google.com/admin-sdk/directory/v1/reference/users/list#try-it>
* <http://oauth2client.readthedocs.io/en/latest/source/oauth2client.service_account.html>
* <https://github.com/google/oauth2client/issues/401>
* <https://github.com/google/oauth2client/issues/418>
* <https://github.com/google/oauth2client/pull/420>

## Method

```python
import httplib2
import secure_storage
from apiclient.discovery import build
from oauth2client.service_account import ServiceAccountCredentials

client_email = '...@developer.gserviceaccount.com'
scopes = [
    'https://www.googleapis.com/auth/admin.directory.user.readonly',
    'https://www.googleapis.com/auth/admin.directory.group.readonly',
    'https://www.googleapis.com/auth/apps.groups.settings'
]
uname = '...@example.com'
fname = '/.../private_key.p12'

creds = ServiceAccountCredentials.from_p12_keyfile(client_email, fname, scopes=scopes)
delegated_creds = creds.create_delegated(uname)

http_auth = delegated_creds.authorize(httplib2.Http())
service = build('admin', 'directory_v1', http=http_auth)
service.users().list(domain='example.com', maxResults=10, orderBy='email').execute()
service.groups().list(domain='example.com', maxResults=10).execute()
```
