# A semi-automated workflow paradigm for the distributed creation and curation of expert annotations
{:.no_toc}

* This will become a table of contents (this text will be scraped).
{:toc}

# Introduction

This page introduces a concrete implementation of the annotation workflow paradigm
proposed by Johannes Hentschel, Fabian Moss, Markus Neuwirth, and Martin Rohrmeier
at the ISMIR conference 2021. It makes use of
[GitHub Actions](https://github.com/features/actions) and can be easily adopted
through the corresponding [template repository](https://github.com/DCMLab/annotation_workflow_template/tree/ismir2021).
It is an adapted version of the implementation that is at the heart of the
[DCML Corpus Initiative](https://www.epfl.ch/labs/dcml/projects/corpus-project/)
and, without further configuration, works out of the box for public repositories
only.

## Use Case

The implementation is designed for a distributed setting where one or several
annotators make use of the [DCML harmony annotation standard](https://github.com/DCMLab/standards)
for entering harmony, phrase, and cadence annotations directly into
uncompressed [MuseScore 3](https://musescore.org/) files (MSCX)
stored in a GitHub repository. It uses commands provided by the parsing library
[ms3](https://pypi.org/project/ms3/) for performing automated tasks, namely

* `ms3 extract` to extract and store information from the annotated MSCX files
  in the form of tab-separated values (TSV) files, namely
  * annotations
  * notes
  * measures
  * metadata
  * (the command allows for extracting additional information)
* `ms3 check` to detect syntax errors in the annotated MSCX files
* `ms3 compare` to store, after a review of annotations, a copy of each reviewed
  MSCX file in which the reviewer's changes are colour-highlighted.

## How to use the implementation

1. Head to the [template repository](https://github.com/DCMLab/annotation_workflow_template/tree/ismir2021)
   and click on "Use this template".
   ![Use this template button](img/use_this_template.png)
1. Create the new repository (if you want it "Private", you need to reconfigure
   the bot, see below).
   ![Create repository](img/create_repo)

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
