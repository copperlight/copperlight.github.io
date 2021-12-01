
## Build

[![Release](https://github.com/copperlight/copperlight.github.io/actions/workflows/release.yml/badge.svg)](https://github.com/copperlight/copperlight.github.io/actions/workflows/release.yml)

## Development

1. [Install Python](https://copperlight.github.io/python/install-the-latest-python-versions-on-macosx/).

1. Set up the local virtual environment.

    ```shell
    ./setup-venv.sh
    ```

1. Activate the virtual environment and serve the site.

    ```
    source venv/bin/activate
    mkdocs serve
    open http://localhost:8000
    ```

## Deployment

* Opening a PR will run the GitHub action to build the site.
* Pushing changes to the `main` branch will trigger the GitHub action to build and deploy the site.

## Known Issues

The [mkdocs-markdownextradata-plugin](https://github.com/rosscdh/mkdocs-markdownextradata-plugin/)
does not handle the Ansible pages well, because they embed Jinja expressions and there does not
appear to be a way to escape those characters in that context. 
