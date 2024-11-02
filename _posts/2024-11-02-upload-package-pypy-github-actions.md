---
title: Upload a package to PyPI automatically with GitHub Actions
date: 2024-11-02 00:00:01 +0000
categories: [python, pypi]
tags: [programming, python, pypi,github,github actions, howto]
img_path: /assets/img/posts/
---

Some time ago, I wrote the post [Upload a package to PyPI](https://rubenhortas.github.io/posts/upload-package-pypi/), but, how everything evolves, this time I decided to evolve and take advantage of all the power of [GitHub Actions](https://docs.github.com/en/actions).

# GitHub actions

[GitHub actions](https://docs.github.com/en/actions) allows us to automate, customize and execute development workflows right in our github repositories. 
With [GitHub actions](https://docs.github.com/en/actions) we can automate a lot of tasks (build, tests, deployments, code reviews, issue management...) when an event occurs in our repository.
[GitHub actions](https://docs.github.com/en/actions) it's a very powerful tool, I have to say that, for me, it has been like discovering a new world.

To automate our tasks, [GitHub](https://github.com) will suggest us some workflows based on the repository type, we can create our own workflows, but we can use workflows created by other users using the [Actions Marketplace](https://github.com/marketplace?type=actions) and we can modfify workflows done by other users (advantages of the open source :P).

In my case, I want to automate the publishing of my project [fail2bangeolocation](https://github.com/rubenhortas/fail2bangeolocation) to [testpypi.org](https://test.pypi.org/) and [pypi.org](https://pypi.org/) under certain events.
The first thing I though about was taking advantage of some workflow written by the community.
But, after reviewing some workflows in [GitHub](https://github.com) and the [Actions Marketplace](https://github.com/marketplace?type=actions), I saw that they didn't quite fit what I wanted, and I decided to write my own workflow based on some of them.

# Publishing using GitHub actions

## Requirements

* A package with the required structure

  To publish our python package to [pypi.org](https://pypi.org/), we'll need to adhere to a specific project structure.
  The structure can vary a bit, but, there is a basic example of the required structure:

  ```shell
  /project
  ├── LICENSE
  ├── pyproject.toml
  ├── README.md
  ├── requirements.txt
  ├── src
  │   ├── project
  │   │   ├── module
  │   │   │   └──...
  │   │   └──...
  │   └──...
  └── tests
      └──...
  ```

* [testpypi.org](https://test.pypi.org/) and [pypi.org](https://pypi.org/) accounts

  Having a [testpypi.org](https://test.pypi.org/) it's optional, but it's recomended.
  Uploading the package first to [testpypi.org](https://test.pypi.org/), we can check if there are problems, and we can check how the upload went before going into production (publishing in [pypi.org](https://pypi.org/)).

# Configuring trusted publishing

Trusted publishing exchanges short-lived tokens (using [OpenID Connect (OIDC)](https://openid.net/developers/how-connect-works/)) between a trusted third-party service (in our case [GitHub](https://github.com)) and [testpypi.org](https://test.pypi.org/) and/or [pypi.org](https://pypi.org/).
Trusted publishing method can be used in automated environments and eliminates the need to use manually generated API tokens to authenticate with [testpypi.org](https://test.pypi.org/) and/or [pypi.org](https://pypi.org/) when publishing.

The steps to crete a [pending] publisher in [testpypi.org](https://test.pypi.org/) and [pypi.org](https://pypi.org/), are basically the same.

> I had the project upload to [testpypi.org](https://test.pypi.org/) and [pypi.org](https://pypi.org/), but If you don't have your project uploaded yet, you can create a ["pending" trusted publisher](https://docs.pypi.org/trusted-publishers/creating-a-project-through-oidc/).
{: .prompt-info}

## Configuring trusted publishing in testpypi

We go to our [testpypi.org](https://test.pypi.org/) account, we go to "*Your projects*" and we select our project (in my case *[fail2bangeolocation](https://github.com/rubenhortas/fail2bangeolocation)*), and we click in "*Manage*":

![Manage](pypi-your-projects.png){: width="700" height="600" .shadow}
*testpypi manage your project*

We go to "*Publishing*":

![Publishing](pypi-publishing.png){: width="700" height="600" .shadow}
*testpypi publishing*

Under "*Trusted Publisher Management*" we add a new publisher:

![New publisher](testpypi-new-publisher.png){: width="700" height="600" .shadow}
*testpypi new publisher*

Here we will set the owner, the repository name, the workflow name and the environment name:

* **Owner**: Our github user.
* **Repository name**: Our github repository name.
* **Workflow name**: The file name of our (future) workflow, here I will use "**publish.yml**".
* **Enviromnment name**: The name of the GitHub Actions environment that the workflow will use, here I will use "**testpypi**".

> We have to pay attention to the workflow and environment names.
> We will need them later.
{: .prompt-warning}

Click on "*Add*" and we are done! ;)

> It should be noted that, many times, we will not be able to install the package generated in testpypi in on our machine (or it may not work) due to dependency problems.
> This is because the same packages (or the same versions) may not be available in testpypi as in pypi.
{: .prompt-info}

## Configuring trusted publishing in pypi:

We go to our [pypi.org](https://pypi.org/) account, we go to "*Your projects*" and we select our project (in my case *[fail2bangeolocation](https://github.com/rubenhortas/fail2bangeolocation)*), and we click in "*Manage*":

![Manage](pypi-your-projects.png){: width="700" height="600" .shadow}
*pypi manage your project*

We go to "*Publishing*":

![Publishing](pypi-publishing.png){: width="700" height="600" .shadow}
*pypi publishing*

Under "*Trusted Publisher Management*" we add a new publisher:

![New publisher](pypi-new-publisher.png){: width="700" height="600" .shadow}
*pypi new publisher*

Here we will set the owner, the repository name, the workflow name and the environment name:

* **Owner**: Our github user.
* **Repository name**: Our github repository name.
* **Workflow name**: The file name of our (future) workflow, here I will use "**publish.yml**".
* **Enviromnment name**: The name of the GitHub Actions environment that the workflow will use, here I will use "**pypi**".

> We have to pay attention to the workflow and environment names.
> We will need them later.
{: .prompt-warning}

Click on "*Add*" and we are done! ;)

# Adding the workflow

> The worflow file name has to match the name we gave it when we created the trusted trusted-publishers
{: .prompt-warning}

GitHub CI/CD workflows are declared in YAML files stored in the `.github/workflows/` directory of our repositories.
So, we can create a `.github/workflows/publish.yml` file, or, we can create it from the [GitHub web interface](https://github.com).

To create the action from the [GitHub web interface](https://github.com), we go to our repository and select "*Actions*":

![GitHub Actions](fail2bangeolocation-actions.png){: width="700" height="600" .shadow}
*GitHub actions*

Now, we will see some suggested actions for our repository, but, this time, we will create our own clicking in "*set up a workflow yourself*":

![Set a workflow yourself](get-started-with-github-actions.png){: width="700" height="600" .shadow}
*Get started with GitHub Actions - Set up a workflow yourself*

In the editor, we have to create a workflow **with a file name that matches the name we gave it when we created the trusted-publishers**:

![New workflow](github-actions-new-workflow.png){: width="700" height="600" .shadow}
*New workflow*

We write (or paste) our workflow, we commit the changes and we are ready to publish!

# The workflow

> The environment names have to match with the enviroment names we gave them wen we created the trusted-publishers.
{: .prompt-info}

You can see my workflow here: [fail2bangeolocation publish.yml](https://github.com/rubenhortas/fail2bangeolocation/blob/main/.github/workflows/publish.yml), but I'm going to copy it and explain it (briefly) here below:

```yml
name: Publish Python distribution 🐍📦

# The workflow is triggered when events taht include that includes tagas starting with v*.*.* or test_v*.*.*. are sent to the repository.
on:
  push:
    tags:
      - 'v*.*.*'
      - 'test_v*.*.*'

jobs:
# Builds the pythond distribution (wheel and source tarball)
  build:
    name: Build distribution 📦
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4 # Checkout the code
    - name: Set up Python
      uses: actions/setup-python@v5 # Set up the Python environment
      with:
        python-version: "3.x"
    - name: Install pypa/build # Install the build tool
      run: >-
        python3 -m
        pip install
        build
        --user
    - name: Build a binary wheel and a source tarball
      run: python3 -m build # Build the package
    - name: Store the distribution packages # Temporarily store the build artifacts in the dist directory under the name python-package-distributions
      uses: actions/upload-artifact@v4
      with:
        name: python-package-distributions
        path: dist/

# Publishes the built package to TestPyPI (for testing) if the tag starts with test_v.*
  publish-to-testpypi:
      name: Publish to TestPyPI 🐍📦 
      needs:
        - build # Start the job only if the build job has completed
      runs-on: ubuntu-latest
  
      environment:
        name: testpypi # Enter the environment name set in the Publisher
        url: https://test.pypi.org/p/example-package-hanaosan0318 # Project URL
  
      permissions:
        id-token: write  # Grant Publishing permissions
  
      if: startsWith(github.ref, 'refs/tags/test_v') # Conditional check for TestPyPI publishing
  
      steps:
      - name: Download all the dists # Download the build artifacts that were saved earlier
        uses: actions/download-artifact@v4
        with:
          name: python-package-distributions
          path: dist/
      - name: Publish distribution 📦 to TestPyPI # Publish to TestPyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          repository-url: https://test.pypi.org/legacy/

Publishes the built package to PyPI (production) if the tag starts with v.*
  publish-to-pypi:
      name: Publish to PyPI 🐍 📦
      needs:
        - build # Start the job only if the build job has completed
      runs-on: ubuntu-latest
      
      environment:
        name: pypi # Enter the environment name set in the Publisher
        url: https://pypi.org/p/example-package-hanaosan0318 # Project URL
        
      permissions:
        id-token: write
  
      if: startsWith(github.ref, 'refs/tags/v') # Conditional check for PyPI publishing
  
      steps:
      - name: Download all the dists
        uses: actions/download-artifact@v4
        with:
          name: python-package-distributions
          path: dist/
      - name: Publish distribution 📦 to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1

  github-release:
      name: Create GitHub Release with source code 📦
      needs:
        - publish-to-pypi # Start the job only if the PyPI publishing job has completed
      runs-on: ubuntu-latest
  
      permissions:
        contents: write # Grant permission to create a GitHub release
  
      steps:
      - name: Checkout code
        uses: actions/checkout@v4 # Checkout the code
  
      - name: Create GitHub Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # A temporary token that is automatically generated each time the workflow is run
        run: >-
          gh release create
          '$\{{ github.ref_name }\}' # '${{ github.ref_name }}' this line is now showed on the textblock
          --repo '${{ github.repository }}'
          --notes "Release for version ${{ github.ref_name }}"
```

## publish.yml workflow TL;DR

This workflow automates building, publishing (to [testpypi.org](https://test.pypi.org/) and [pypi.org](https://pypi.org/)), and creating a [GitHub](https://github.com) release for your Python package.

This workflow is triggered when a version tag is pushed.
The tag will be a label that match the patterns `test_v*.*.*` or `v*.*.*.`.

This workflow will do a (slightly) different job depending on the type of tag:

* If a `test_v*.*.*` tag (e.g. `test_v1.0.1`) is pushed, a test version will be generated.
The workflow will build the distribution packages and upload them to [testpypi.org](https://test.pypi.org/).

* If a `v*.*.*` tag (e.g. `v1.0.1`) is pushed, the relase version will be generated.
The workflow will build the packages, upload them to [pypi.org](https://pypi.org/) **and generate the GitHub release**.

## Why I decided to write my own workflow? 

The reason why I decided to write my own workflow is because the workflows I saw depended on other events, and, usually, they did a lot of work together (as publish to [testpypi.org](https://test.pypi.org/), [pypi.org](https://pypi.org/) and generate the [GitHub](https://github.com) release at the same time).
I prefer a slightly more structured workflow, based on pushing version tags. And, I want to perform slightly different actions based on these tags.

Publishing the test version first in [testpypi.org](https://test.pypi.org/) let me know if there are any errorsm and let me know the result of the distribution files.
Once I have verified that the upload to [testpypi.org](https://test.pypi.org/) is correct, and the distribution files have been checked, I can generate the production version, upload it to [pypi.org](https://pypi.org/) and generate the [GitHub](https://github.com) release.

## Why I wanted to publish based on tags?

When you upload a package to [testpypi.org](https://test.pypi.org/) or [pypi.org](https://pypi.org/), you can't repeat versions.
With this workflow I can create a `dev` branch to add changes and upload test versions to [testpypi.org](https://test.pypi.org/).
In case there are errors when uploading to [testpypi.org](https://test.pypi.org/), I can increase the project version, until everything is correct, without affecting the production version number.
Once everything is correct I can merge the `dev` branch changes into the `main` branch, tag the production version in `main`, and the production release will be generated automatically (ideally without errors).
Once the release version is published I will take care of cleaning up the versions in [testpypi.org](https://test.pypi.org/), and, if necessary, in [pypi.org](https://pypi.org/).

*Enjoy! ;)*
