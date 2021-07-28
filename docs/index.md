# First heading

## Steps occurring in each job

### Cloning the repository to perform actions on it

Before any action can be performed on the files in your repository, it needs
to be cloned on the virtual machine. This is usually done via the action

  - name: Checkout main
    uses: actions/checkout@v2
    with:
      path: main

## Configuring the bot for pushing commits

By default, the workflow implementation uses GitHub's standard bot. In your
history it will figure under the name `github-actions` and, without further
configuration (see below), it only works on public repositories.
