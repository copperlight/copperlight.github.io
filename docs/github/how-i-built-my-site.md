# How I Built My Site

<div class="meta">
  <span class="date"><small>2021-12-01</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

I recently rebuilt my GitHub Pages site, switching my tech stack to [Python], [MkDocs] and
[Material for MkDocs]. The previous workflow had too many moving parts and the theme was hard to
read.  It turns out that MkDocs has some nice automation hooks for GitHub Pages and it looks great
on mobile browsers with the [Material] theme. Since I use MkDocs for site generation at work, I am
already pretty familiar with it.

Writing documents in [Markdown] and turning them into web pages with a static site generator is the
fastest and easiest way to post articles on your GitHub Pages site.  This format allows you to
drop down into HTML, when necessary, to enhance your page formatting, but you shouldn't need to do
this for most pages.

[Material]: https://material.io/
[Material for MkDocs]: http://squidfunk.github.io/mkdocs-material/
[MkDocs]: http://www.mkdocs.org/
[Python]: https://www.python.org/
[Markdown]: https://daringfireball.net/projects/markdown/syntax

## Install Python and MkDocs Packages

If you are relying on a system Python, consider following these instructions:
[Install the Latest Python Versions on Mac OSX].

Install [Material for MkDocs]. This package will bring along all the necessary dependencies,
including [MkDocs], for a fully functioning site.

```bash
pip install mkdocs-material
```

[Install the Latest Python Versions on Mac OSX]: ../python/install-the-latest-python-versions-on-macosx.md

## Sign Up for Google Analytics

!!! note
    You must disable ad-blocking software in order to be able to see the Google Analytics page.

1. Navigate to [Google Analytics](https://analytics.google.com/analytics/web/) > Admin
1. Property > Create New Property
    1. **Account Name:** $YOUR_ACCOUNT_NAME
    1. **Website Name:** username.github.io
    1. **Website URL:** https://username.github.io
    1. Go with the default configuration options - you can change these later.
    1. Get Tracking ID
1. Note the Tracking ID (looks like: `UA-00000000-0`) assigned to this property.

## Build the Site

1. [Create a new repository](https://github.com/new) named `$USERNAME.github.io`, where `$USERNAME`
is your username (or organization name) on GitHub.  It must match exactly, or it will not work.

2. Clone the repository locally. Requires [SSH keys](https://help.github.com/articles/connecting-to-github-with-ssh/).

        git clone git@github.com:$USERNAME/$USERNAME.github.io.git
        cd $USERNAME.github.com

3. Create a directory structure that looks like the following:

        $USERNAME.github.io/
        ├── .github/
        │   └── workflows/
        │       ├── pr.yml
        │       └── release.yml
        ├── .gitignore
        ├── docs/
        │   ├── css/
        │   │   └── custom.css
        │   └── index.md
        ├── mkdocs.yml
        └── requirements.txt

4. Create a GitHub Actions configuration file for pull requests (`./.github/workflows/pr.yml`). This
will build the site for every PR that is submitted.

        name: pull-request
        
        on: [pull_request]
        jobs:
          build:
            runs-on: ubuntu-latest
            steps:
              - uses: actions/checkout@v2
              - uses: actions/setup-python@v2
                with:
                  python-version: 3.x
              - run: pip install -r requirements.txt
              - run: mkdocs build

5. Create a GitHub Actions configuration file for deploying the site (`./.github/workflows/release.yml`),
replacing `$USERNAME` with your username.

        name: release
        
        on:
          push:
            branches:
            - main
        jobs:
          deploy:
            if: ${{ github.repository == '$USERNAME/$USERNAME.github.io' }}
            runs-on: ubuntu-latest
            steps:
              - uses: actions/checkout@v2
              - uses: actions/setup-python@v2
                with:
                  python-version: 3.x
              - run: pip install -r requirements.txt
              - run: git fetch origin gh-pages:gh-pages
              - run: mkdocs gh-deploy

7. Create a `./mkdocs.yml` site configuration file. Choose a [Creative Commons license] for your
site - I chose [CC BY-NC-SA 4.0] for my site. Add your Google Analytics property Tracking ID to
`extra.analytics`. If you do not have a Tracking ID, then delete these lines in the configuration
file.

        site_name: My Site
        site_url: 'http://$USERNAME.github.io/'
        repo_url: 'https://github.com/username/$USERNAME.github.io'
        edit_uri: ''
        site_description: My Site
        site_author: My Name
        copyright: <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png" /></a>

        strict: True

        pages:
        - Home: index.md

        theme: material

        extra:
          analytics:
            provider: google
            property: UA-00000000-0

        extra_css:
        - css/custom.css

        markdown_extensions:
        - admonition
        - codehilite
        - pymdownx.tilde
        - toc:
            permalink: True

[Creative Commons license]: https://creativecommons.org/choose/
[CC BY-NC-SA 4.0]: http://creativecommons.org/licenses/by-nc-sa/4.0/

1. Create a `./docs/index.md` file.

        Hello World!

2. Create a `./docs/css/custom.css` file.

        /* Avoid showing first header, i.e., the page title in the sidebar.
         *
         * https://github.com/mkdocs/mkdocs/issues/318#issuecomment-98520139
         */
        li.toctree-l3:first-child {
          display: none;
        }

        code {
          font-family: Menlo, monospace;
        }

       .meta .date { float: left }
       .meta .discuss { float: right }

3. Create a `./requirements.txt` file, so that you can easily reinstall the necessary Python packages
with `pip install -r requirements.txt`.

        mkdocs-material

4. Create a `./.gitignore` file.

        site/
        venv/

5. Build and serve the site locally, to verify that your changes look good.  When MkDocs is up and
running, you will see `Serving on http://127.0.0.1:8000`. Leave this process running. When you are
done developing and testing your site, you can stop this process with `ctrl+c`. Open a new Terminal
to run the second command.

        mkdocs serve
        open http://localhost:8000

6. Push the changes to GitHub to save your progress. The first release build will fail due to the
absence of the `gh-pages` branch.

        git add --all
        git commit -m "first commit"
        git push origin

7. Build and deploy the site for the first time. This step is done to establish the `gh-pages`
branch which is used by GitHub Actions. The static HTML site generated by this process will be
pushed to the `gh-pages` branch at the origin for this repository. Changes will be live within
one minute. Navigate to the site url to see your changes.

        mkdocs gh-deploy

## Repo Configuration

Navigate to the repository on GitHub and set some useful configuration options.

1. Code > Description
1. Code > Website
1. Code > Manage topics
1. Settings > Branches > Protected branches: `main`
    1. Check: Protect this branch
    1. Check: Include administrators

## New Post Workflow

1. Create a new branch, so you can check your work in a PR.

        git checkout -b new-post

2. Start serving the site locally, with file change detection.

        mkdocs serve
        open http://localhost:8000

3. Start a new post by creating a markdown file in the `./docs` directory hierarchy.

4. Images can be served from a location such as `./docs/images`, with references as follows:

        ![Link Name](/images/my-file.png "Alt Text")

5. Add the new markdown file to the `./mkdocs.yml` site configuration and continue editing. See
[Writing Your Docs](http://www.mkdocs.org/user-guide/writing-your-docs/) for tips on arranging
your Markdown files.

6. When editing is complete, commit changes and push. Open a PR on the GitHub site and check
the build output. Merge when you are happy with the changes and the release build will deploy
the updated site.

        git add --all
        git commit -m "my new post"
        git push origin
