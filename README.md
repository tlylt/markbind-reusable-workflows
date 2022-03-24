# markbind-reusable-workflows
A list of GitHub Reusable Workflows that help improve your CI/CD with a MarkBind site.
- Create a new workflow to preview proposed changes to your MarkBind site from forks PRs
## Option Summary

### fork-build

Option        | Required |                      Default | Remarks
:-------------|:--------:|-----------------------------:|----------------------------------------------
version       |    no    |                     'latest' | The MarkBind version to use to build the site
rootDirectory |    no    |                          '.' | The directory to read source files from
baseUrl       |    no    | Value specified in site.json | The base URL relative to your domain
siteConfig    |    no    |                  'site.json' | The site config file to use

### fork-preview

Option | Required | Default | Remarks
:------|:--------:|--------:|-----------------------------------------
token  |   yes    |         | The token to be used for the service
domain |    no    |      '' | The domain that the site is available at

## Usage
With `fork-build.yml` and `fork-preview.yml`, you can establish a secure workflow that builds your MarkBind site triggered by a fork PR.
Then, the site artifact will be uploaded and deployed for preview. Note that the choice of using two separate workflows is to reduce possible security issues, as [recommended](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/) by the GitHub Security Lab.

In the repository, you will need to add two workflows to `.github/workflows`.
The first workflow being `fork-build.yml` will be triggered by a `push` event.
The second workflow being `fork-preview.yml` will be triggered whenever `fork-build` is completed.

```yaml
name: Build Fork PR

# read-only
# no access to secrets
on:
  pull_request:

concurrency: 
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    name: Build Fork PR
    uses: tlylt/markbind-reusable-workflows/.github/workflows/fork-build.yml@main
    with:
      version: "3.1.1"
```

```yaml
name: Preview Fork PR

on:
  workflow_run:
    workflows: ["Build Fork PR"]
    types: [completed]

jobs:
  on-success:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    name: Deploying to surge
    uses: tlylt/markbind-reusable-workflows/.github/workflows/fork-preview.yml@main
    secrets:
      token: ${{ secrets.SURGE_TOKEN }}
```
