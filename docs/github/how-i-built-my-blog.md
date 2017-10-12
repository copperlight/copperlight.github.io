# How I Built My Blog

<small>2014-06-28</small>

I have settled on [GitHub Pages](https://pages.github.com/) and
[Jekyll Bootstrap](http://jekyllbootstrap.com/) as the fastest way to get started with blogging in
a way that gives me some reasonable control over the formatting and design process.  There are
several things that I like about this combination:

* GitHub handles the source, backups, hosting and scaling
* Jekyll adds the structure to make it viable as a blog (and serves locally for prototyping!)
* Markdown is fast and easy to write for formatting purposes

I am pleased with how easy it is to get started:

```bash
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
cd USERNAME.github.com
git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
git push origin master
```

I had originally started configuring my GitHub Pages repo a few months ago and the code was slightly
out of sync with the JB repo.  I want to be able to pickup changes from the JB upstream master, so I
configured that:

```bash
git remote add upstream git@github.com:plusjade/jekyll-bootstrap.git
git pull upstream master
git push origin
```

In the process of configuring my blog, I found the following links helpful:

* [How I Built My Blog in One Day](http://erjjones.github.io/blog/How-I-built-my-blog-in-one-day/) by Eric Jones
* [Let Me Introduce My Blog](http://log.malchiodi.com/2013/01/19/my-first-post/) by Malchio
* [Jekyll Bootstrap Theme Browser](http://themes.jekyllbootstrap.com/)

Eric's blog has several tricks for integrating with various other web services like social sites and
what not that seems like it would be useful for increasing engagement with posts.

## Configuration Details

I like the modern fonts and white-on-black look of the Hooligan theme, so I added that:

```bash
rake theme:install git="https://github.com/dhulihan/hooligan.git"
```

I fixed a deprecation warning in `_config.yml`, changing `pygments: true` to `highlighter: true`.

The new method of using GitHub Pages [locks the version of Jekyll](https://help.github.com/articles/using-jekyll-with-pages)
through a `Gemfile` specified like so:

```ruby
source 'https://rubygems.org'
gem 'github-pages'
```

This is important, since they are using a fairly old version of Jekyll (1.5.1 versus 2.1.0) and a
few other gems.  From here, you install the github-pages gem which brings in the dependencies:

```bash
gem install bundler
bundle install
```

Following these steps will result in downloading the [specific gem versions](https://pages.github.com/versions/)
used for GitHub pages.  This will make it a lot easier to figure out why pages fail to compile
(email notifications are sent for these failures).  In my case, it turns out that GitHub Pages was
defaulting to using the `maruku` markdown interpreter, which is now end of life, and which was also
failing to compile my markdown.  The solution is to switch to the newer `kramdown` markdown
interpreter by adding the following to `_config.yml`, noting that this field is not part of the
latest JB configuration:

```yaml
markdown: kramdown
```

## Blog Post Development Workflow

The basic post development workflow looks like this:

```bash
# start serving the site locally
bundle exec jekyll serve

# start a new post
rake post title="Hello World"

# write the post

# rebuild the site
bundle exec jekyll build

# review post at http://localhost:4000
```

Typing `jekyll build` frequently becomes old fast, so I added [guard-jekyll](https://github.com/therabidbanana/guard-jekyll)
to the bundle and ran it like so:

```bash
bundle install guard-jekyll
bundle exec guard init jekyll
bundle exec guard
```

This will monitor the directory for file changes and automatically rebuild the site, so I can stay
focused in Sublime Text and refresh the page in the browser.
