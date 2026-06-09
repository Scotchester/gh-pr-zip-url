# gh-pr-zip-url

A [GitHub CLI](https://cli.github.com) extension that takes the number of a PR
and returns the URL to its zip archive.

It works whether the PR is on a branch in the origin repo or on another user's fork.


## Example

From the Scotchester/jellybeans repository:

```sh
$ gh pr-zip-url 19
https://github.com/jgravois/jellybeans/archive/refs/heads/patch-1.zip
```


## Installation

**Prerequisite:** gh-pr-zip-url requires having [jq](https://jqlang.org/download/)
available on your `PATH`.

Install with the `gh extension install` command:

```sh
gh extension install Scotchester/gh-pr-zip-url
```


## Why?

My motivation for creating this command was making it easier to test changes to a
[custom template for the Wagtail CMS](https://docs.wagtail.org/en/stable/reference/project_template.html#using-custom-templates).
The `wagtail start` command can accept a `--template` argument that takes a URL to a zip file,
and I wanted a quicker way to run the command using the code of a contributor's pull request
in order to review it before merging.

With gh-pr-zip-url already installed, adding this function to my `.zshrc`
makes it a single command to generate the project and get it ready for testing:

```zsh
wagtailstartpr() {
    # Start a new Wagtail project using a PR branch on a template repo
    # (e.g., https://github.com/wagtail/news-template)
    #
    # Takes one argument:
    # the number of a PR in a Wagtail project template repo you're currently in

    # Store the current folder name
    TEMPLATE_FOLDER=${PWD:t}

    # Replace characters not allowed in Python identifiers with underscores
    SAFE_NAME=$(echo "$TEMPLATE_FOLDER" | sed -e 's/[^a-zA-Z0-9_]/_/g')

    # Make the module name identifiable and unique
    MODULE_NAME=${SAFE_NAME}_test_pr_"$1"

    # Use gh CLI to get and store PR archive URL
    PR_ZIP_URL=$(gh pr-zip-url "$1")

    # Activate the template repo's virtualenv, if necessary
    source .venv/bin/activate

    # Back out of template repo because we don't want template-ception
    cd ../

    # Run the wagtail start command to generate the project
    wagtail start $MODULE_NAME --template $PR_ZIP_URL

    # Deactivate the template repo's virtualenv
    deactivate

    # Move into the new project's directory
    cd $MODULE_NAME

    # Create a project-specific virtualenv and activate it
    python -m venv .venv
    source .venv/bin/activate

    # Install the requirements
    pip install -r requirements.txt
}
```

Then when it's time to test a PR, I just run `wagtailstartpr 123` and within seconds
I'm ready to inspect the generated code, run the dev server, or run the tests.
