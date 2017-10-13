# pet - CLI Snippet Manager

<small>2017-10-13</small>

<https://github.com/knqyf263/pet>

This shell utility that makes it easy to save, search and recall shell commands.  It uses
[GitHub Gists](https://gist.github.com/copperlight) as the backend storage, so your work is backed
up and accessible from multiple locations.  Of course, since you are storing data publicly, make
sure that you are sanitizing the data, so no secrets are shared.

## Install and Configure

Install with Homebrew:

```
brew install pet
```

[Create a new access token](https://github.com/settings/tokens) on GitHub that allows only the
`gist` scope.  Create a pet configuration file and set the `access_token`.

```
pet configure
```

## Usage

Create new snippets:

```
pet new
```

List existing snippets:

```
pet list
```

Search for snippets:

```
pet search
```

!!! warning
    The `pet` command does not have version validation associated with snippets uploaded to Gist,
    so it is possible to wipe out your snippets in two separate ways:

    1. Downloading empty snippets from your Gist, which overwrite your local snippets.

    1. Uploading empty local snippets to your Gist.  Since GitHub stores revisions, you can recover
    from this.

    Make sure that you have the latest snippets locally before updating your Gist.

Upload snippets to Gist:

```
pet sync -u
```

Download snippets from Gist:

```
pet sync
```
