
## Build

[![Build Status](https://travis-ci.org/copperlight/copperlight.github.io.svg?branch=master)](https://travis-ci.org/copperlight/copperlight.github.io)

## Development

1. [Install Python](https://copperlight.github.io/python/install-the-latest-python-versions-on-macosx/).

1. Set up the local virtual environment.

    ```shell
    ./setup-venv.sh
    ```

1. Activate the virtual environment and serve the site.

    ```
    sourve venv/bin/activate
    mkdocs serve
    open http://localhost:8000
    ``` 

## Deploy Configuration

1. [Generate a new SSH key](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key),
named `travis-deploy-key`. 

    ```shell
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f travis-deploy-key
    ```

1. Add the public key to your GitHub repository as a [Deploy Key](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys),
named `travis-deploy-key` and allow write access. 

1. Install [Travis CLI](https://github.com/travis-ci/travis.rb) and login with GitHub credentials.

    ```shell
    travis login
    travis whoami
    ```

1. Encrypt the private key, using Travis CLI. This will upload the key to Travis and provide a
one-liner for decrypting the key, which should be added to `.travis.yml`.

    ```shell
    travis encrypt-file travis-deploy-key
    ``` 

1. Configure the [Travis build](./.travis.yml).

## Deployment

Pushing changes to the `source` branch will trigger the build and deploy steps on Travis.

## Known Issues

The [mkdocs-markdownextradata-plugin](https://github.com/rosscdh/mkdocs-markdownextradata-plugin/)
does not handle the Ansible pages well, because they embed Jinja expressions and there does not
appear to be a way to escape those characters in that context. 
