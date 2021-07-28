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

## How to use the workflow implementation

### Create a GitHub repository

1. Head to the [template repository](https://github.com/DCMLab/annotation_workflow_template/tree/ismir2021)
   and click on "Use this template".\
   ![Use this template button](img/use_this_template.png)
1. Create the new repository (if you want it "Private", you need to reconfigure
   the bot, see below).\
   ![Create repository](img/create_repo.png)
1. In the new repo, click on "Code" to copy the URL for the `git clone` command
   and clone the repo to your machine (`git clone git@github.com:johentsch/annotated_mscx_files.git`
   in the example here).\
   ![Get clone URL](img/clone_url.png)

### Add uncompressed MuseScore files

1. Create a subfolder (here called `MS3`), add the MSCX (uncompressed MuseScore
   format) files to it, and push everything to GitHub. During the following
   minute, the yellow circle indicates that the `ms3_extract` Action is running.\
   ![ms3_extract action in progress](img/add_extract.png)
1. Refresh the page until the yellow circle becomes a green check. You will
   see that the `github-actions` bot pushed a commit creating the folders
   `measures` and `notes` as well as the files `README.md` and `metadata.tsv`.\
   ![Commit created by the bot](img/after_extract.png)


These files are automatically updated every time an MSCX file is added to or
modified on the main branch. You can add custom text to the README.md as long
as you do it above the 'Overview' heading. Everything below this heading is
automatically overwritten.

### Annotate files

1. Create a new branch, use MuseScore 3 to add annotation labels to one of the
   harmony layers (`Add -> Text -> {Chord Symbol|Roman Numeral Analysis|Nashville Number}`).
   For example, take this annotated _Ecossaise No. 7_, D. 145 by Franz Schubert:\
   ![Annotated Schubert Ecossaise](img/D145ecossaise07.png)
1. Commit the changes. Every time you push to a child branch, the labels in
   all changed files will be checked for syntactical correctness according to
   the [DCML harmony annotation standard](https://github.com/DCMLab/standards):\
   ![Check in progress](img/check_annotations.png)
1. If the yellow circle turns into a red cross, at least one syntax error was
   found and by clicking on it you can have it displayed. In this example, the
   output shows one wrong label in measure 8, onset 1/4 (beat 2), because
   the `}` indicating the phrase ending needs to be the last character:\
   ![Displaying syntax errors](img/syntax_error.png)
1. Once the error is fixed, the new annotations can be merged into the main
   branch by opening a Pull Request:\
   ![Pull Request for an annotated file](img/annotated_pr.png)
1. After merging the Pull Request, the `ms3_extract` action is triggered again
   which will
  * extract a tabular overview of the labels and store it as `harmonies/[file name].tsv`:\
    ![Annotation table](img/annotation_table.png)
  * read out the metadata from the updated MuseScore file which had been modified
    in MuseScore by the annotator like so:\
    ![Updating metadata in MuseScore](img/metadata.png)
  * and writes these metadata to `metadata.tsv` and to the overview in the README file:\
    ![Updated README file](img/updated_readme.png)


Our [Annotation Tutorial](https://dcmlab.github.io/standards/build/html/tutorial/musescore.html#placing-the-annotation-cursor)
has some more explanations on how to conveniently add Roman Numerals in MuseScore 3.

In case you are using a different annotation standard you might want to remove
or replace the DCML syntax check by other code.

### Review files

1. The reviewer checks out a child branch and commits their changes to the
   reviewed MuseScore file, which includes adding their initials to the file's
   metadata so that they will appear in the README upon merge into main.
1. Then, the reviewer creates a pull request to suggest the changes to
   the annotator who is supposed to go through them to approve or contest them:\
   ![Pull request opened by reviewer](img/reviewed_pr.png)\
   This time, because of the presence of the `harmonies/[file name].tsv`
   file, the `label_comparison` results in the `github-actions` bot pushing
   an auxiliary MSCX file highlighting the changes made by the reviewer:\
   ![Reviewed Schubert Ecossaise](img/D145ecossaise07_reviewed.png)

# Documentation

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

The installation happens in the following three steps:

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

In the subsequent steps, the commands of the ms3 library can be called.

## Configuring the bot

By default, the workflow implementation uses GitHub's standard bot for pushing
files to your repository. In the Git history, its commits will figure under
the name `github-actions` and, without further configuration (see below),
it only works on public repositories.

# Known limitations

* PRs with over 100 commits
* deleting MuseScore files
* MuseScore files with > 50 MB (rare)
