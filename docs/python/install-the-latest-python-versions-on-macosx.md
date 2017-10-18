# Install the Latest Python Versions on Mac OSX

<div class="meta">
  <span class="date"><small>2017-10-12</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

Once you have decided that Python is an [awesome language to learn](https://automatetheboringstuff.com/)
and you have heard all about the the [cool features](https://www.youtube.com/watch?v=9zinZmE3Ogk)
awaiting you, you are determined to install the latest versions on your laptop so that you can start
developing.

* [What’s New In Python 3.6](https://docs.python.org/3.6/whatsnew/3.6.html) - December 23, 2016
* [What’s New In Python 3.5](https://docs.python.org/3.5/whatsnew/3.5.html) - September 13, 2015
* [What’s New In Python 3.4](https://docs.python.org/3.4/whatsnew/3.4.html) - March 16, 2014
* [What’s New In Python 3.3](https://docs.python.org/3.3/whatsnew/3.3.html) - September 29, 2012
* [What’s New In Python 3.2](https://docs.python.org/3.2/whatsnew/3.2.html) - February 20, 2011
* [What’s New In Python 3.1](https://docs.python.org/3.1/whatsnew/3.1.html) - June 27, 2009
* [What’s New In Python 3.0](https://docs.python.org/3.0/whatsnew/3.0.html) - December 3, 2008
* [What’s New In Python 2.7](https://docs.python.org/2.7/whatsnew/2.7.html) - July 3, 2010
* [What’s New In Python 2.6](https://docs.python.org/2.6/whatsnew/2.6.html) - October 1, 2008

You may be a little concerned about the differences between Python 2 and Python 3, so you want
to make sure that you have the latest versions of each available. This will enable you to switch
between them easily and work on porting Python 2 programs to Python 3.

* [Should I use Python 2 or Python 3 for my development activity?](https://wiki.python.org/moin/Python2orPython3)
* [Key Differences between Python 2.7 and Python 3 with Examples](http://sebastianraschka.com/Articles/2014_python_2_3_key_diff.html)
* [Porting Python 2 Code to Python 3](https://docs.python.org/3/howto/pyporting.html)
* [Writing code that runs under both Python2 and 3](https://wiki.python.org/moin/PortingToPy3k/BilingualQuickRef)

OS vendor Python versions lag behind the latest available and can only be updated by installing a
major patch or bolting-on an additional Python installation:

<table>
    <tr> <th>OS Version <th>System Python Version
    <tr> <td>OSX 10.13 <td>2.7.10
    <tr> <td>OSX 10.12 <td>2.7.10
    <tr> <td>OSX 10.11 <td>2.7.10
    <tr> <td>Ubuntu 16.04 LTS Xenial Xerus <td><a href="https://packages.ubuntu.com/xenial/python">2.7.11</a>
    <tr> <td>Ubuntu 14.04 LTS Trusty Tahr <td><a href="https://packages.ubuntu.com/trusty/python">2.7.5</a>
    <tr> <td>Ubuntu 12.04 LTS Precise Pangolin <td><a href="https://ubuntuforums.org/showthread.php?t=1969449">2.7.3</a>
</table>

One way to work around this issue is to use [`pyenv`](https://github.com/pyenv/pyenv) to install the
versions of Python you want to use.  This tool provides a simplified build environment for installing
the Pythons you want, while providing a set of shell script shims that makes it easy to switch between
them.  This tool was inspired by and forked from [`rbenv`](https://github.com/rbenv/rbenv) and
[`ruby-build`](https://github.com/rbenv/ruby-build).

## Install Python with pyenv

1. Start a Terminal session.

1. Install [Homebrew](https://brew.sh/) and Xcode Command Line Tools.

        xcode-select --install

        /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

1. Install `pyenv`.

        brew install pyenv

1. If you already have `pyenv` installed, you can upgrade it to gain access to the latest Python
versions.  The `pyenv` tool is updated periodically with new formulas for the latest releases.

        brew upgrade pyenv

1. Add configuration to your `.bash_profile` to initialize `pyenv` every time you start a new
Terminal. The `PYENV_ROOT`, which is where Python versions and packages will be installed,
defaults to `$HOME/.pyenv`.

        cat >>$HOME/.bash_profile <<"EOF"
        if which pyenv > /dev/null 2&>1; then eval "$(pyenv init -)"; fi
        EOF

1. Load your `.bash_profile` configuration changes.

        source $HOME/.bash_profile

1. List the available Python versions.  Notice that you have access to several different
distributions: python.org (plain version numbers), anaconda, ironpython, jython, miniconda,
pypy and stackless.  We will install the standard Python versions released by
[python.org](https://www.python.org/downloads/), otherwise known as CPython, because the
interpreter is written in C.

        pyenv install --list

1. If you are running OSX 10.13, you will need to set the following environment variables, when you
install new Python versions. See [#988](https://github.com/pyenv/pyenv/issues/988) for more details.

        export CFLAGS="-I$(brew --prefix openssl)/include"
        export LDFLAGS="-L$(brew --prefix openssl)/lib"

1. Install Python versions.

        pyenv install 2.7.14
        pyenv install 3.6.3

1. List the available Python versions.

        pyenv versions

1. Activate a Python version and verify that it is available.

        pyenv global 2.7.14
        pyenv versions
        python -V

1. Activate another Python version and verify that it is available.

        pyenv global 3.6.3
        pyenv versions
        python -V

1. If desired, activate multiple Python versions and verify that they are available.
[PEP 394 -- The "python" Command on Unix-Like Systems](https://www.python.org/dev/peps/pep-0394/)
explains conventions for naming `python` binaries.

        pyenv global 2.7.14 3.6.3
        pyenv versions
        python -V
        python2 -V
        python2.7 -V
        python3 -V
        python3.6 -V

1. Create new directories for Python projects, add `pyenv local` version files and verify the Python
versions.  Your `python` version will automatically switch when you change into these directories.

        mkdir python2-project
        cd python2-project
        pyenv local 2.7.13
        cat .python-version
        pyenv local
        python -V
        cd ..

        mkdir python3-project
        cd python3-project
        pyenv local 3.6.3
        cat .python-version
        pyenv local
        python -V
        cd ..

## Useful Python Packages

This is a small collecton of useful packages that will help get you started doing useful things
with Python.

Install [flake8](http://flake8.pycqa.org/en/latest/), which is a static analyzer that enforces
good Python coding style and alerts you to coding mistakes.  Now that you have Python installed,
you can use the `pip` command to install any additional modules that you want to use.  You will
need to install modules separately for each version of Python you are actively using.

```
pip install flake8
```

Install advanced Python Read-Eval-Print-Loop (REPL) packages, which will make it easier to explore
Python code, because they grant access to tab completion and fancy editing capabilities.  The `ipython`
tool is a command line REPL, which can be exited with `ctrl-d`.  Jupyter Notebook is a browser-based
REPL that operates on individual cells of code.  The [IPython Interactive Computing](https://ipython.org/)
website has more information on these tools.

```
pip install ipython jupyter
ipython
jupyter notebook
```

Install [Requests: HTTP for Humans](http://docs.python-requests.org/en/master/), which makes it easy
to consume HTTP services. See GitHub's [REST API v3](https://developer.github.com/v3/) documentation
for more details on endpoints that are avaialble.

```
pip install requests
```

```
python
>>> import requests
>>> r = requests.get('https://api.github.com/users/copperlight')
>>> r.ok
True
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
u'{"login":"copperlight", ...}'
>>> r.json()
{u'public_repos': 27, ...}
>>> r.json()['login']
u'copperlight'
```

Install [Flask: A Python Microframework](http://flask.pocoo.org/), which can be used to quickly build
small websites that automate everyday tasks.

```
pip install Flask
```

```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"
```

```
FLASK_APP=hello.py flask run
curl http://127.0.0.1:5000
```

If you need Python package isolation on a per-project basis, because you have conflicting sets
of packages specified for different projects, you can use [virtualenv](https://virtualenv.pypa.io/en/stable/)
to set up separate environments.  The `virtualenv` tool establishes a clean room Python installation,
either based upon the current version of Python in use, or a version specified on the command line.
If you have multiple Python versions activated in `pyenv`, you can use `virtualenv` to switch between
them easily.

```
pip install virtualenv
pip list

mkdir python2-project
cd python2-project
virtualenv -p python2.7 venv
source venv/bin/activate
python -V
which python
pip list
which pip
deactivate
cd ..

mkdir python3-project
cd python3-project
virtualenv -p python3.6 venv
source venv/bin/activate
python -V
which python
pip list
which pip
deactivate
cd ..
```

## Automated Testing with Python

While working on Python programs, you may be interested in automating testing.  The most common
choice for implementing this is [tox](https://tox.readthedocs.io/en/latest/index.html), which will
allow you to run tests with multiple different versions of Python, if needed.

## Python Packaging

After completing a Python program or two, you will be interested in learning how to distribute them
effectively.  The [Python Packaging User Guide](https://packaging.python.org/) has some good advice
on this topic.

## Quickstart Commands

```bash
# if you need homebrew
xcode-select --install
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# install pyenv and latest major python versions
brew install pyenv

cat >>$HOME/.bash_profile <<"EOF"
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
EOF

source $HOME/.bash_profile

pyenv install 2.7.14
pyenv install 3.6.3
pyenv versions
```
