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

## Option Details

### token
Token for Surge.sh
- Example: `${{ secrets.SURGE_TOKEN }}`
  - `SURGE_TOKEN` is the environment secret name
- Require registration with an email address
- After retrieving the token, put the token as a repository secret in your
- See [here](https://markbind.org/userGuide/deployingTheSite.html#previewing-prs-using-surge) for a detailed guide on how to retrieve the token

### domain
The domain that the site is available at. Surge.sh allows you to specify a subdomain as long as it is not taken up by others.

- 'xxx.surge.sh'
  - A typical domain that you can specify. You have to ensure that 'xxx' is unique. Read [here](https://surge.sh/help/adding-a-custom-domain) on how to configure a custom domain with Surge.sh
  - Note that for PR Preview purposes, the domain will be prefixed with 'pr-x', which 'x' is the GitHub event number
  - E.g. 'pr-1.xxx.surge.sh' where 'xxx.surge.sh' is what you specified as the domain

### version
The MarkBind version to use to build the site.
- 'latest'
  - This is the latest published version of MarkBind
- 'master'
  - This is the latest, possibly unpublished version of MarkBind
- 'X.Y.Z'
  - This is the version of MarkBind with the specified version number
  - A sample version number is '3.1.1'

### rootDirectory (MarkBind CLI arguments)
The directory to read source files from.
- '.'
  - This is the default value
  - This is for the case that your source files of the MarkBind site are in the root directory of the repository
- './path/to/directory'
  - This is for the case that your source files of the MarkBind site are in a subdirectory of the repository
  - A sample path is './docs'

### baseUrl (MarkBind CLI arguments)
The base URL relative to your domain.
- '/reponame'
  - Defaults to the value of `baseUrl` in your `site.json` file
  - Note that you will need to specify this or in the site config file (typically the `site.json`), if you wish to configure the relative URL correctly.

### siteConfig (MarkBind CLI arguments)
The site config file to use.
- 'site.json'
  - This is the default value
## Usage
With `fork-build.yml` and `fork-preview.yml`, you can establish a secure workflow that builds your MarkBind site triggered by a fork PR.
Then, the site artifact will be uploaded and deployed for preview. Note that the choice of using two separate workflows is to reduce possible security issues, as [recommended](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/) by the GitHub Security Lab.

In your repository, you will need to add two workflows to `.github/workflows`.
- The first workflow being `fork-build.yml` will be triggered by a pull request.
- The second workflow being `fork-preview.yml` will be triggered whenever `fork-build` is completed.

```yaml
name: Build Fork PR

# read-only
# no access to secrets
on:
  pull_request:

# cancel multiple runs at the same time, can be removed if you don't need it
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
    with:
      domain: "mb-test/surge.sh"
    secrets:
      token: ${{ secrets.SURGE_TOKEN }}
```
