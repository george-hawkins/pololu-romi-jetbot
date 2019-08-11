Pip basics with Python 3 on Ubuntu 16.04
========================================

It seems impossible to upgrade pip, even locally, without also breaking the system pip:

    $ sudo apt install python3-pip
    $ pip3 --version
    pip 8.1.1 from /usr/lib/python3/dist-packages (python 3.5)
    $ python3 -m pip install --user --upgrade pip
    $ python3 -m pip --version
    pip 19.2.1 from ~/.local/lib/python3.5/site-packages/pip (python 3.5)
    $ pip3 --version
    Traceback (most recent call last):
      File "/usr/bin/pip3", line 9, in <module>
        from pip import main
    ImportError: cannot import name 'main'

I've added a comment to this [SO answer](https://stackoverflow.com/a/50205248/245602) asking why this is.

It turns out it's infinitely easier to just bootstrap a few things and then do everything else via `venv`:

    $ sudo apt install python3 python3-pip python3-venv
    $ pip3 --version
    pip 8.1.1 from /usr/lib/python3/dist-packages (python 3.5)
    $ python3 -m venv env
    $ source env/bin/activate
    $ pip install --upgrade pip 
    $ pip --version
    pip 19.2.1 from .../env/lib/python3.5/site-packages/pip (python 3.5)
    $ pip install --upgrade setuptools wheel

Note that `venv`, used as above, takes the name given to it, i.e. `env` in this case, and creates a subdirectory, with that name, in your current directory.

So the intention is that you create such environments on a per-project basis. When you move from one project to another you need to activate the environment for the current project, i.e. use `source ...` as above.

Any packages that you install, while a particular environment is active, will end up somewhere below the base directory corresponding to that environment.

The last step above, installing `setuptools` and the more modern `wheel`, are needed if you later end up installing something that comes as a source archives. If you don't install them, `pip` will still be able to install most things (pre-built binary archives) and will fail fairly clearly in the situations where the `setuptools` or `wheel` package are needed.

While in a particular environment `pip` and `python` are bound to the versions associated with that environment, i.e. you don't need to explicitly use `python3` or `pip3`.

    $ ls env/bin
    activate  activate.csh  activate.fish  easy_install  easy_install-3.5  pip  pip3  pip3.5  python  python3  wheel
    $ type python
    pip is .../env/bin/python
    $ type pip
    pip is .../env/bin/pip
    $ /usr/bin/env python
    >>> import sys
    >>> print(sys.executable)
    .../env/bin/python
    >>> exit()

The last command, i.e. `/usr/bin/env python`, shows that `python` used in a shebang will also evaluate to the environment's version of Python.

To leave an environment altogether and just return to the system setup:

    $ deactivate 
    $ pip --version
    pip 8.1.1 from /usr/lib/python2.7/dist-packages (python 2.7)

Pyenv
-----

Some packages can be pretty dependent on a particular version of Python, e.g. the _latest_ versions of [`compiledb`](https://pypi.org/project/compiledb/) require Python 3.6.

You can install PPAs for different versions as per this [AskUbuntu answer](https://askubuntu.com/questions/865554/how-do-i-install-python-3-6-using-apt-get).

But [`pyenv`](https://github.com/pyenv/pyenv) allows you to easily install Python versions locally and combined with `venv` seems a better solution.

Install `pyenv` using the [installer](https://github.com/pyenv/pyenv-installer):

    $ curl https://pyenv.run | bash

And (as it tells) you add the following three lines to `~/.bashrc`:

    export PATH=$HOME/.pyenv/bin:$PATH
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"

Then source `~/.bashrc` or open a new terminal tab and...

    $ pyenv versions
    * system (set by ~/.pyenv/version)
    $ pyenv install 3.6.9

You can find the latest Python version for a given series, 3.5.X, 3.6.X etc., [here](https://www.python.org/downloads/).

If the install complains about missing dependencies, you can work out what's up from the [`pyenv` wiki](https://github.com/pyenv/pyenv/wiki/common-build-problems).

Now set the installed version as the version to use for this session:

    $ pyenv local 3.6.9
    $ python --version
    Python 3.6.9
    $ type python
    python is ~/.pyenv/shims/python
    $ type python3
    python3 is ~/.pyenv/shims/python3

Now setup a `venv`:

    $ python -m venv env
    $ source env/bin/activate
    $ pip install --upgrade pip setuptools wheel

Once you've got a `venv` setup, you don't need to worry about or use `pyenv` again - when you activate the environment in future you'll get the Python version that you'd specified, using `pyenv`, when the environment was created.

    $ ls -l env/bin/python
    lrwxrwxrwx 1 joebloggs joebloggs 7 Aug 10 18:14 env/bin/python -> python3
    $ ls -l env/bin/python3
    lrwxrwxrwx 1 joebloggs joebloggs 48 Aug 10 18:14 env/bin/python3 -> ~/.pyenv/versions/3.6.9/bin/python3
    $ pyenv local 3.6.9
    
Now you can install and use whatever package it was the depended on that particular Python version:

    $ pip install compiledb
