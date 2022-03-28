# markbind-reusable-workflows
A list of GitHub Reusable Workflows that help improve your CI/CD with a MarkBind site.
- Create a new workflow to preview proposed changes to your MarkBind site from forks PRs
## Option Summary

### fork-build

Option                                                 | Required |                      Default | Remarks
:------------------------------------------------------|:--------:|-----------------------------:|----------------------------------------------
[version](#version)                                    |    no    |                   `'latest'` | The MarkBind version to use to build the site
[rootDirectory](#rootdirectory-markbind-cli-arguments) |    no    |                        `'.'` | The directory to read source files from
[baseUrl](#baseurl-markbind-cli-arguments)             |    no    | Value specified in site.json | The base URL relative to your domain
[siteConfig](#siteconfig-markbind-cli-arguments)       |    no    |                `'site.json'` | The site config file to use

### fork-preview

Option            | Required | Remarks
:-----------------|:--------:|-----------------------------------------
[token](#token)   |   yes    | The token to be used for the service
[domain](#domain) |   yes    | The domain that the site is available at

## Option Details

### version
The MarkBind version to use to build the site.
- Latest
  - `'latest'`
  - This is the latest published version of MarkBind
- Development
  - `'development'`
  - This is the latest, possibly unpublished version of MarkBind in development
- Any valid version
  - `'X.Y.Z'`
  - This is the version of MarkBind with the specified version number
  - E.g. `'3.1.1'`
- Any valid version range
  - Internally the action calls [`npm install`](https://docs.npmjs.com/cli/v6/commands/npm-install) to install the specified version of MarkBind
  - Hence, a version range such as `'>=3.0.0'` (or semantic versioning like `'^3.1.1'`) is also valid

### rootDirectory (MarkBind CLI arguments)
The directory to read source files from.
- Root
  - `'.'`
  - This is the default value
  - This is for the case that your source files of the MarkBind site are in the root directory of the repository
- Any subdirectory
  - `'./path/to/directory'`
  - This is for the case that your source files of the MarkBind site are in a subdirectory of the repository
    - E.g. `'./docs'`
### baseUrl (MarkBind CLI arguments)
The base URL relative to your domain.
- Default
  - The value of `baseUrl` in the site config file (typically `site.json`)
- Any valid base URL
  - For GitHub Pages, you will need to specify this here or in the site config file, in order to configure the relative URL correctly.
    - e.g. `'/reponame'`
  - For Surge, you will need to ensure that it's specfied as `''` here or in the site config file.

### siteConfig (MarkBind CLI arguments)
The site config file to use.
- Default site config file
  - `'site.json'`
  - This is the default value
- Name of your site config file
  - If your site config file is not named `site.json`, specify the name here
    - E.g. `'ug-site.json'`

### token
Token for Surge.sh
  - `${{ secrets.SURGE_TOKEN }}`
    - `SURGE_TOKEN` is the environment secret name
  - Require registration with an email address
  - After retrieving the token, put the token as a repository secret in your repository
  - See [here](https://markbind.org/userGuide/deployingTheSite.html#previewing-prs-using-surge) for a detailed guide on how to retrieve the token

### domain
The domain that the site is available at.
- A surge.sh subdomain
  - `'<subDomain>.surge.sh'`
  - Surge allows you to specify a subdomain for free as long as it has not been taken up by others. You have to ensure that the `<subDomain>` is unique. 
  - A possible subdomain to use is your repository name: e.g. `mb-test.surge.sh`
- Additional notes
  - for PR preview purposes, the domain you specify will automatically be prefixed with 'pr-x-', where 'x' is the GitHub event number
    - E.g. `'pr-x-<domain>'` (and hence `'pr-1-mb-test.surge.sh'`)
  - Custom domain does not work with PR preview
  - This action will not automatically cleanup merged PR deployments. Follow this [instruction](https://surge.sh/help/tearing-down-a-project) to manually tear down the deployed site if required

## Usage
With `fork-build.yml` and `fork-preview.yml`, you can establish a secure workflow that builds your MarkBind site triggered by a fork PR.
Then, the site artifact will be uploaded and deployed for preview. Note that the choice of using two separate workflows is to reduce possible security issues, as [recommended](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/) by the GitHub Security Lab.

In your repository, you will need to add two workflows to the `.github/workflows` folder.
- The first workflow being `fork-build.yml` will be triggered by a pull request.
- The second workflow being `fork-preview.yml` will be triggered whenever `fork-build` is completed.

```yaml
name: Build Fork PR

# read-only
# no access to secrets
on:
  pull_request:
    branches:
      - main

# cancel multiple runs at the same time, can be removed if you don't want it
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
    name: Deploying to Surge
    uses: tlylt/markbind-reusable-workflows/.github/workflows/fork-preview.yml@main
    with:
      domain: "mb-test.surge.sh"
    secrets:
      token: ${{ secrets.SURGE_TOKEN }}
```
