# Python Tutorial with Git, Gitlab-CI, Cloudbuild-CI, Docker, Gcloud

Disclaimer: I have simplified and overlooked details in this guide.

This tutorial assumes you have read the [Python101 slides](https://docs.google.com/presentation/d/1WdBEU1BSDfEKHkLxY2IksiaAqXJBSKPUc_0Q4UJAtIY/edit):

* Python 3 is properly installed
* There are no other tools interferring with the python setup
* Steps that require admin permission will be explicitly marked.

Limitations of the tutorial:

* Because on how python is setup, we will be limited to running one python 2 version and one python 3 version.

  * After finishing this tutorial, if running several python 3 versions is required, I recommend to have a look on [Pyenv](https://github.com/pyenv/pyenv)
  * If running windows, I'm afraid you will need to resort to conda.
* After thinking about it for a good while, I don't want to explain everything from the very beginning. If in doubt, I try to follow the Zen of python:

```text
[I] (test-project) javier@sam ~/t/test-project (master)> python -c 'import this'
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

## FAQ: Why would I want to read this?

This guide will explain how python works as an execution environment, where the different directories and files that matter sit, and what are good ways of working in python.

The idea is that this knowledge will allow you to be confident that you won't be breaking your computer, and that you will understand how the different commands you run affect your installation.

## Understanding the python path

`import sys` will trigger a lookup in the so called Python Path, looking for a package or module with that name to import.

Our ability to import modules greatly depends on this feature. The Python path is similar to the shell PATH variable.

In all the systems, the PATH variable marks where to look for binaries. Therefore, running `base64`, will look on all those places, and the first match will be the one executed.

In the case of python, same thing happens with modules. Doing `import sys` will trigger the same behaviour of traversing accross the different paths looking for the `sys` package or module.

## Understanding the global environment

Default python installations should be clean from any kind of external packages. The only packages I recommend to put in the global namespace are the ones to create virtual environments. More specifically, I use `pipenv`, therefore the following output would be in the global environment because I ran `pip3 install pipenv` (but for linux)

```text
root@6c4b4dd258dd:/$ pip freeze
certifi==2019.3.9
pipenv==2018.11.26
virtualenv==16.4.3
virtualenv-clone==0.5.2
```

By default, `python` will use the global environment, which is installed along python. When speaking about environment, I refer to the `site-packages` folder.

Python installation can be divided in 2:

* Packages that are part of the core of python (such as `os`, `sys`, `time`)
* Packages that are external to the python language project (such as `pipenv`, `virtualenv`, `numpy`)

When choosing installations, the core of python will be setup in the python version folder, where as the other packages will be setup in the `site-packages` folder.

Example output of the environment:

```text
root@6c4b4dd258dd:/# python -c 'import sys;print("\n".join(sys.path))'

/usr/local/lib/python37.zip
/usr/local/lib/python3.7
/usr/local/lib/python3.7/lib-dynload
/usr/local/lib/python3.7/site-packages
```

As we can see in the printed paths, there are quite a few folders and files there. Let's have a look on them.

Although I'm going to explain what these directories contain, I have sorted the directories by relevance marking 2 of them as optional.

### Script directory or current directory

It may be difficult to realise, but that empty line at the beginning of the python paths means current directory. Python will add the location of the python script we execute to the python path automatically. This is quite confusing when using the path of an script inside a package.

Have a look at [https://github.com/txomon/python-paths](https://github.com/txomon/python-paths) for some examples

### Core python installation

In this case `/usr/local/lib/python3.7` is the place where all python is installed:

```
root@6c4b4dd258dd:/# ls /usr/local/lib/python3.7/ | tail
datetime.py
...
os.py
...
sys.py
```

As we can see, the typical first level import modules are present there.

### Site packages

Site packages is the name of the folder where we store all python packages and modules that are not developed and distributed by the python organization.

In this case `/usr/local/lib/python3.7/site-packages` is the system site-packages folder.

As we can see, all the packages that were installed for pipenv are available here:

```text
root@6c4b4dd258dd:/# ls /usr/local/lib/python3.7/site-packages/
README.txt                   pkg_resources
__pycache__                  setuptools
certifi                      setuptools-40.8.0.dist-info
certifi-2019.3.9.dist-info   virtualenv-16.4.3.dist-info
clonevirtualenv.py           virtualenv.py
easy_install.py              virtualenv_clone-0.5.2.dist-info
pip                          virtualenv_support
pip-19.0.3.dist-info         wheel
pipenv                       wheel-0.33.1.dist-info
pipenv-2018.11.26.dist-info
```

Later we will probably dive-in into what are all those files there that don't look ly python modules. But for now let's just keep in mind that these files e

### (Advanced) Compiled modules

When we speak about compiled modules in python, we usally can refer to python byte-code or to dynamic libraries. In this specific case, we are referring to the later.

One of the ways to improve the performance of our Python programs is through the implementation in a low level language of some parts of the routines that require performance. One of the most common ones are C and C++.

Python has an API to create our own python modules coded in C, so that we can import those modules from normal python scripts, however running in the low level implementation we coded.

```text
root@6c4b4dd258dd:/# ls /usr/local/lib/python3.7/lib-dynload/ | head
_asyncio.cpython-37m-x86_64-linux-gnu.so
_bisect.cpython-37m-x86_64-linux-gnu.so
_blake2.cpython-37m-x86_64-linux-gnu.so
_bz2.cpython-37m-x86_64-linux-gnu.so
_codecs_cn.cpython-37m-x86_64-linux-gnu.so
_codecs_hk.cpython-37m-x86_64-linux-gnu.so
_codecs_iso2022.cpython-37m-x86_64-linux-gnu.so
_codecs_jp.cpython-37m-x86_64-linux-gnu.so
_codecs_kr.cpython-37m-x86_64-linux-gnu.so
_codecs_tw.cpython-37m-x86_64-linux-gnu.so
```

Here, we can see in the list asyncio module's low level implementation.

### (Advanced) Zip python core installation

Python supports distributing the sourcecode in zipfiles. This is something that usually leads to easier distribution, but it's really easy to get wrong.

In general zipped python packaging is outside the scope of this guide, but feel free to read about it at [PEP-273](https://www.python.org/dev/peps/pep-0273/)

## Understanding a virtual environment

We have explained how python looks up packages and modules, and we have seen how the global python installation is organized.

This may not be an obvious thing, but in software development, dependencies are many times a big headache. There are many ways in which the different languages sort out and use dependencies. In python, we try to separate each application to run with its own environment. These is achieved making use of the python path and virtual environments.

A virtual environment to put it simple is just a new location that is inserted earlier in the path than the global system installation discussed earlier.

### Installing pipenv

Although I have said it implicitly, from this part on we will need to have pipenv installed. To do so, with no virtualenv activated and admin priviledges run `pip install pipenv`.

If using linux, your distribution probably has packaged pipenv as `python-pipenv` or something similar. In OSX, I would discourage installing python packages through brew, it works somewhat weirdly, and we just want one package in the global `site-packages` + a binary in the bin folder.

### Creating a virtual environment with pipenv

The main creation tool to create just virtual environments is `venv` or `virtualenv`. However, in our case we want to have a few commodities over those two tools.

Traditionally, on top of those two, we can find `virtualenv-wrapper`, which takes care of the location. In our case, we are going to use `pipenv` because besides the aforementioned functionalities, `pipenv` gives a few nice extras around dependency management.

At the beginning of the guide I showed how I only have globally installed a few packages. These are all `pipenv` and its dependencies. I do not recommend installing pipenv from brew.

Additionally, this guide will only use `pipenv`, not show how to use it. The full documentation is available in [the official documentation page](https://pipenv.readthedocs.io/en/latest/)

We will create virtual environments using `pipenv shell`, which will launch a new shell with the environment activated.

```text
root@c88f7f192fde:~# mkdir test-project
root@c88f7f192fde:~# cd test-project/
root@c88f7f192fde:~/test-project# pipenv shell
Creating a virtualenv for this project…
Pipfile: /root/test-project/Pipfile
Using /usr/local/bin/python (3.7.3) to create virtualenv…
⠧ Creating virtual environment...Already using interpreter /usr/local/bin/python
Using base prefix '/usr/local'
New python executable in /root/.local/share/virtualenvs/test-project-Ph4xz48u/bin/python
Installing setuptools, pip, wheel...
done.

✔ Successfully created virtual environment!
Virtualenv location: /root/.local/share/virtualenvs/test-project-Ph4xz48u
Creating a Pipfile for this project…
Launching subshell in virtual environment…
in/activate192fde:~/test-project#  . /root/.local/share/virtualenvs/test-project-Ph4xz48u/bi
(test-project) root@c88f7f192fde:~/test-project#
```

### Virtualenv paths

We know we have a virtual environment activated because there are those parenthesis at the beginning of the shell line `(test-project)`.

Also, if we run the same command as before, we will see that the python path has been modified

```text
(test-project) root@c88f7f192fde:~/test-project# python -c 'import sys;print("\n".join(sys.path))'

/root/.local/share/virtualenvs/test-project-Ph4xz48u/lib/python37.zip
/root/.local/share/virtualenvs/test-project-Ph4xz48u/lib/python3.7
/root/.local/share/virtualenvs/test-project-Ph4xz48u/lib/python3.7/lib-dynload
/usr/local/lib/python3.7
/root/.local/share/virtualenvs/test-project-Ph4xz48u/lib/python3.7/site-packages
```

As before, we have the current directory the first, and we can see that besides the global python installation, the global site-packages directory has dissappeared, and has been replaced by the one inside the virtualenv.

This way, we can have several virtual environments, with different packages and versions, where each virtual environment is installing the packages in its own `site-packages` directory.

Feel free to install new packages using pipenv, and check how the `site-packages` directory gets populated with new packages and modules.

## About directories and working environments

Before starting off with the first project, I would consider a few things that usually developer come across on their own:

* Spaces in directories or files are forbidden. They cause a ton of trouble and we end up paying the price way later when changing paths it's practically impossible
* Create a directory to hold all our code. Having projects in the desktop is handy, but code ends up getting mixed with downloads and what not, and that gets unmanageable quickly. Here there are a few approaches:
  * Directory in the root called "projects" or "code" (`/projects/`)
  * Directory in HOME (`/home/javier/projects/`), supposing the path doesn't contain any space, called "projects" or "code"
  * Same thing as before, but organising the code in subfolders (per project for example)

## Creating a python project

The first thing we do to create a python project is to create a directory for the project. Recommendations on naming are the typical, use lowercase, dash separated names.

I like to do the key steps through the terminal, and I very much recommend learning how to do everything through the terminal before switching to some kind of interface.

```text
root@c88f7f192fde:~# mkdir test-project
root@c88f7f192fde:~# cd test-project/
```

Once inside, we can start a pipenv environment, and head over to the IDE.

```text
root@c88f7f192fde:~/test-project# pipenv shell
Creating a virtualenv for this project…
Pipfile: /root/test-project/Pipfile
Using /usr/local/bin/python (3.7.3) to create virtualenv…
⠧ Creating virtual environment...Already using interpreter /usr/local/bin/python
Using base prefix '/usr/local'
New python executable in /root/.local/share/virtualenvs/test-project-Ph4xz48u/bin/python
Installing setuptools, pip, wheel...
done.

✔ Successfully created virtual environment!
Virtualenv location: /root/.local/share/virtualenvs/test-project-Ph4xz48u
Creating a Pipfile for this project…
Launching subshell in virtual environment…
in/activate192fde:~/test-project#  . /root/.local/share/virtualenvs/test-project-Ph4xz48u/bi
(test-project) root@c88f7f192fde:~/test-project#
```

### Opening a folder as a project with Pytharm

We can now open this folder with pycharm. Depending on the setup, pycharm may be available from the command line. But for sake of simplicitly let's just assume we open it graphically.

It is really important to keep in mind that opening a file is not the same as opening a folder (AKA project). When we open a file with Pycharm, it's the same as opening the file with notepad++. It will colour the words, but it will not work as an IDE.

When we open a folder with Pycharm, it automatically tries to index all the contents of the folder, and generate autocomplete and other features. In some cases it will autodetect the virtual environment, and link our imports to the real packages.

![Opening a folder with pycharm](images/01-open-project.png)

We will see that pycharm automatically starts working and doing things (we can check this in the bottom status bar, on the right side)

The first thing to do when we open a new folder with pycharm is to check what is the python interpreter is using. They call it python interpreter as a way to group a virtualenv, remote python installations and the global local installation.

For that, we can head to `Settings -> Project: project-name -> Project Interpreter`

![Settings section](images/02-project-interpreter.png)

Depending on our luck (because many times is really about luck), pycharm will automatically detect the virtualenv pipenv created (we know if it's the one we want by reading in the dropdown box where it says `Project Interpreter`). If the virtual environment , or if we are not using pipenv, we may need to select it ourselves.

For this, there is an small wheel icon on the right side of the `Project interpreter` dropdown box. Click it and choose `Add`.

![Adding manually the virtual environment](images/03-add-interpeter.png)

This will pop up a new window that will give us all those options I mentioned before.

In our specific case, we will always head to the section `Pipenv Environment`.

There, we will pick the global python as `Base interpreter`, In my case `/usr/bin/python3`. Please make sure to specify the major version of python here, don't use `python` but rather `python3` or `python2`.

If this is selected, or if we see a red message like the one in the screenshot, we click OK (if possible) or Cancel, meaning that the setup was already done.

![Pipenv section](images/04-pick-pipenv-base-python.png)

#### (Advanced) Selecting external virtualenv

If we were adding a virtualenv that is not through `pipenv`, would head to the first section, `Virtualenv Environment`, and pick the virtualenv from either the list, or browsing to `~/.virtualenvs/projectname-xxxxx`.

![Select virtualenv directly](images/05-pick-virtualenv.png)

### Coding our first script

There are steps that can be done out of order, and we could create if we wanted all the scaffolding of the project first, but I just hate overcomplicating things from the beginning, and I don't want to do it now.

I always start coding in a single script file that I call `main.py`, and for now, we will be coding everything there.

TODO
----
### Initializing a git repository

Each project should run in it's own git repository. Having more than one project per git repository will give us infinite headaches in the future, so the recommendation is to avoid it by having everything cleanly separated.

To create a git repository, run `git init`

```
[I] (test-project) javier@sam ~/t/test-project> git init
Initialized empty Git repository in /home/javier/tmp/test-project/.git/
```

Before continuing, let's setup git

#### Setting up git

There are two main things that I always setup when using git. First one is to make sure that my email is correct and my name is the same everywhere. Second one is to setup the global gitignore.

`.gitignore` is the file that is usually in the repository to ignore files, however, the `.gitignore` shipped in the repos should only ignore files that are a sideeffect of the project. **The fact that we use one editor or another should be unknown to the project**

The global gitignore is the file `~/.config/git/ignore`, if the folder doesn't exist, just create it with `mkdir -p ~/.config/git/`. The contents vary between different people.

In my case, these are the files that I ignore:

```text
[I] (test-project) javier@sam ~/t/test-project> cat ~/.config/git/ignore
/.idea/
/.vscode/
*.orig
*.new
```

I mainly use IntelliJ base IDEs (By JetBrains), and I'm starting to use visual studio code (By microsoft, but opensource) more lately. The other two lines refer to files when there is a git merge collision.
