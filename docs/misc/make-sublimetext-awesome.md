## Introduction

I was using Atom for awhile, but it's slow (3+ second) startup times started to get to me.  Once I learned the magic of cmd+shift+p from Atom (inspired by the Sublime Text Command Palette), I went back to the original and figured out how to get it configured nicely.  One of the good things about this approach is that you end up with a reasonable Python IDE when you are done.  Also works nicely for Clojure - the REPL has good support for selecting and pasting code snippets from the editing window.  Comparing the two briefly:

| Application | Pros | Cons |
|-------------|------|------|
| Atom | Faster to get started. You end up with a prettier and functional setup with less configuration effort. <br> Package manager is batteries included. <br> Markdown preview rendering in the application. <br> Treeview colors files that have been modified in a Git repo. | Super slow (3-10 second) application load times. <br> Python code completion still a work in progress. <br> No support for treeview directory flattening. |
| SublimeText | Super fast (0-1 second) application load times. <br> Python Jedi code completion. <br> Supports directory flattening into Java package names, although a bit weird. <br> Python console for programmatic configuration. <br> Multi-language REPL in the application. | Package Control is not part of the standard distribution. <br> The ST2/ST3 schism, although ST3 seems to be in better shape these days. <br> Markdown preview rendering forks a browser process, although it keeps track of the rendered temporary file and can reload it. <br> No colorized flagging in treeview for modified Git repo files. |

## Install and Configure

### Install pyenv

!!! warning 
    When you use pyenv with Sublime Text and SublimeLinter-flake8, it will attempt to use python3 from the /usr/local/var/pyenv/shims directory, regardless of your global configuration. You must have flake8 installed with the latest version of python3 on your system in order for the plugin to work.

Uses [Homebrew](http://brew.sh/); this makes it easy to switch between Python2 and Python3.

```shell
xcode-select --install

brew update
brew install pyenv readline

cat >> $HOME/.bash_profile <<"EOF"
export PYENV_ROOT=/usr/local/var/pyenv
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
EOF

source $HOME/.bash_profile
pyenv install --list
pyenv install 2.7.10
pyenv install 3.5.0

pyenv versions
pyenv global 2.7.10
pip install pip --upgrade
```

### Install leiningen

```shell
brew install leiningen
```

### Install Application

Install [Sublime Text 3](http://www.sublimetext.com/3).

Create the following symlinks so you can launch SublimeText from the command line:

```shell
ln -nsf "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
 
# open current directory in sublimetext to see a treeview
subl .
```

### Install Packages

Install Package Control (ctrl+\`):

```python
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

Install the following packages (cmd+shift+p: Package Control: Install Package).

| Package | Purpose |
|---------|---------|
| [BracketHighlighter](https://packagecontrol.io/packages/BracketHighlighter) | Matches a variety of brackets such as: [], (), {}, "", '', #!xml <tag></tag>, and even custom brackets. |
| [EditorConfig](https://packagecontrol.io/packages/EditorConfig) | Standardize editor configurations for projects. |
| [Enhanced Clojure](https://packagecontrol.io/packages/Enhanced%20Clojure) | Improved code completion for Clojure. |
| [FileDiffs](https://packagecontrol.io/packages/FileDiffs) | Shows diffs between the current file, or selection(s) in the current file, and clipboard, another file, or unsaved changes. |
| [fish-shell](https://packagecontrol.io/packages/fish-shell) | Syntax highlighting and whatnot for fish-shell. |
| [Git](https://packagecontrol.io/packages/Git) | Git integration. |
| [GitGutter](https://packagecontrol.io/packages/GitGutter) | Show an icon in the gutter area indicating whether a line has been inserted, modified or deleted. |
| [Gradle_Language](https://packagecontrol.io/packages/Gradle_Language) | Gradle language support. |
| [Jedi - Python autocompletion](https://packagecontrol.io/packages/Jedi%20-%20Python%20autocompletion) | Plugin to the awesome autocomplete library [Jedi](https://github.com/davidhalter/jedi). |
| [lispindent](https://packagecontrol.io/packages/lispindent) | Properly indents lisp code. |
| [Markdown Preview](https://packagecontrol.io/packages/Markdown%20Preview) | Preview and build your markdown files quickly in your web browser, supports Python Markdown Parser and Github Markdown.
| [Origami](https://packagecontrol.io/packages/Origami) | You tell Sublime Text where you want a new pane, and it makes one for you. It works seamlessly alongside the built-in layout commands.
| [ShellCommand](https://packagecontrol.io/packages/ShellCommand) | The ShellCommand plugin allows arbitrary shell commands to be run and their output to be sent to buffers or panels.
| [SublimeLinter](https://packagecontrol.io/packages/SublimeLinter) | A framework for interactive code linting in the Sublime Text 3 editor.
| [SublimeLinter-contrib-scalastyle](https://packagecontrol.io/packages/SublimeLinter-contrib-scalastyle) | This linter plugin for [SublimeLinter](http://sublimelinter.readthedocs.org/) provides an interface to [scalastyle](http://www.scalastyle.org/). It will be used with files that have the “scala” syntax.
| [SublimeLinter-flake8](https://packagecontrol.io/packages/SublimeLinter-flake8) | This linter plugin for SublimeLinter provides an interface to [flake8](http://flake8.readthedocs.org/en/latest/). It will be used with files that have the “Python” syntax.
| SublimeLinter-jshint | This linter plugin for [SublimeLinter](http://sublimelinter.readthedocs.org/) provides an interface to [jshint](http://www.jshint.com/docs/). It will be used with files that have the “JavaScript” syntax, or within \<script
> tags in HTML files. |
| [SublimeLinter-shellcheck](https://packagecontrol.io/packages/SublimeLinter-shellcheck) | This linter plugin for [SublimeLinter](http://sublimelinter.readthedocs.org/) provides an interface to [shellcheck](http://www.shellcheck.net/about.html). It will be used with files that have the “Shell-Unix-Generic” syntax (aka Shell Script (Bash)). |
| [SublimeREPL](https://packagecontrol.io/packages/SublimeREPL) | Run an interpreter inside ST2 (Clojure, CoffeeScript, F#, Groovy, Haskell, Lua, MozRepl, NodeJS, Python + virtualenv, R, Ruby, Scala...).
| [Theme - Soda](https://packagecontrol.io/packages/Theme%20-%20Soda) | Dark and light custom UI themes for Sublime Text.
| [View In Browser](https://packagecontrol.io/packages/View%20In%20Browser) | Open whatever is in your current view/tab. If the file current open is new and has not been saved a temporary file is created (in your default temp directory for your OS) with the extension of .htm and your browser will open it. 
| [Trailing Spaces](https://packagecontrol.io/packages/TrailingSpaces) | Highlights trailing spaces (and lets you delete them in a flash!) |

### Install Package Helpers

In order for some packages to work properly, they need helper binaries to be installed separately, such as:

```shell
brew install scalastyle shellcheck
pip install flake8 jedi
npm install -g jshint
```

### Configure SublimeText

There are a number of configuration tweaks that you will want to make to SublimeText to improve overall behavior.  The easiest way to manage this is to establish a dotfiles repository and track changes in source control.  See the subl directory for a collection of configuration files; the init-dots.sh script describes how these are linked to the main configuration location on your machine.

After installing the configuration files, quit and restart SublimeText - this is particularly necessary for SublimeLinter to function correctly.

### Automate Package Installation

http://stackoverflow.com/questions/19529999/add-package-control-in-sublime-text-3-through-the-command-line

## Editing Tips

SublimeText has some interesting multi-editing features. See also http://docs.sublimetext.info/en/latest/reference/keyboard_shortcuts_osx.html.

| Shortcut | Action |
|----------|--------|
| ctrl+g | Go to line number. |
| cmd+d | Select the next match of the current selection. |
| cmd+ctrl+g | Select all matches of the current selection. |
| cmd+l | Select the next line. |
| cmd+shift+j | Select all child elements within a structured (e.g. HTML) document. |
| cmd+r | Jump to function. |
| cmd+shift+d | Duplicate current line, preserving all indentation. |
| cmd+ctrl+up | Move line up. |
| cmd+ctrl+down | Move line down. |
| cmd+k,b | Hide or show the sidebar. |
| cmd+click | Create multiple cursors. |
| cmd+p | Search for file within project. |
| cmd+alt+left | Go to left tab. |
| cmd+alt+right | Go to right tab. |
ctrl+shift+m	Select text within brackets (expandable).