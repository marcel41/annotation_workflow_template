# First heading

## Steps occurring in each job

### Cloning the repository to perform actions on it

Before any action can be performed on the files in your repository, it needs
to be cloned on the runner, i.e. on the virtual machine. This is usually done
via the action [checkout](https://github.com/actions/checkout).

    - name: Checkout main
      uses: actions/checkout@v2
      with:
        path: main

* The `path` directive clones the repo into the new folder `main`.
* If you want to use the workflow implementation on a private repo, you need to
  configure a bot token (see below).
* If you want to perform actions on a particular branch, you can use the `ref`
  directive (for an example, see the next section)

### Installing the Python library `ms3`

The [ms3](https://pypi.org/project/ms3/) library provides the commands used
by this workflow implementation to parse MuseScore files and perform the
actions `check`, `extract`, and `compare`. Therefore it needs to be installed
on the runner. Instead of the latest version of the library (installable
via `pip install ms3`), this implementation uses a dedicated version which
lies in the branch `workflow` of the code repository.

The installation is done through the following three steps:

    - name: Set up Python 3.8
      uses: actions/setup-python@v1
      with:
        python-version: 3.8
    - name: Clone ms3
      uses: actions/checkout@v2
      with:
        repository: johentsch/ms3
        ref: workflow
        path: ./ms3
    - name: Install ms3
      run: |
        python -m pip install --upgrade pip
        python -m pip install -e ./ms3

## Configuring the bot

By default, the workflow implementation uses GitHub's standard bot for pushing
files to your repository. In the Git history, its commits will figure under
the name `github-actions` and, without further configuration (see below),
it only works on public repositories.
