---
title: Upload a package to PyPI
date: 2022-07-22 00:00:01 +0000
categories: [python, pypi]
tags: [programming, python, pypi, howto]
---

Although the [Packaging Python Projects tutorial](https://packaging.python.org/en/latest/tutorials/packaging-projects/) is very detailed in how to package a Python Project, I encountered some difficulties at the time of do it, so I'm going to leave here a summary and some notes to my future me ;)

First of all, if we want made our application available as a command-line tool, we need to define an [entry point](https://packaging.python.org/en/latest/specifications/entry-points) in our pyproject.toml:

```
[project.scripts]
projectname = "projectname.projectname:main"
```

## Creating accounts

We will need two accounts:

* [test.pypy.org](https://test.pypi.org/account/register/)
* [pypi.org](https://pypi.org)

## Updating the tools

We will need to have installed the last version of *build* and *twine*:

```shell
python3 -m pip install --upgrade build
```

```shell
python3 -m pip install --upgrade twine
```

## Building

Run this command from the same directory where pyproject.toml is located:

```shell
python3 -m build
```

## Configuring the access to [test.pypi.org](https://test.pypi.org)

We will need to create a PyPI API token to securely upload our project. We can create one at [https://test.pypi.org/manage/account/#api-tokens](https://test.pypi.org/manage/account/#api-tokens)

To avoid having to copy and paste the token every time you upload, we can create a $HOME/.pypirc file:

```
[distutils]
  index-servers =
    testpypi
    projectname

[testpypi]
  username = __token__
  password = pypi-token

[fail2bangeolocation]
  repository = https://test.pypi.org/legacy/
  username = __token__
  password = pypi-token
```

## Uploading the package to testpypi

```shell
python3 -m twine upload --repository testpypi dist/*
```

## Checking test package installation

```shell
python3 -m pip install --index-url https://test.pypi.org/simple/ projectname
```

If everything went well...

## Uploading the package to PyPI

```shell
python3 -m twine upload dist/*
```

## Checking package installation

```shell
pip3 install projectname
```

Sources: 

* [Packaging Python Projects](https://packaging.python.org/en/latest/tutorials/packaging-projects)  
* [Entry points specification](https://packaging.python.org/en/latest/specifications/entry-points)  
* [Metadata Hatch](https://hatch.pypa.io/latest/config/metadata)

_Enjoy! ;)_
