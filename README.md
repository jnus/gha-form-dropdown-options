# Issue Forms Dropdown Options

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Dropdown%20Options-blue.svg?colorA=24292e&colorB=0366d6&style=flat&longCache=true&logo=github)](https://github.com/marketplace/actions/issue-forms-dropdown-options)
[![Sponsor ShaMan123](https://img.shields.io/badge/Sponsor%20%E2%9D%A4%20-ShaMan123-%E2%9D%A4?logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/ShaMan123)

[![🧪 Test](https://github.com/ShaMan123/gha-form-dropdown-options/actions/workflows/test.yml/badge.svg)](https://github.com/ShaMan123/gha-form-dropdown-options/actions/workflows/test.yml)
[![🚀 Update Bug Report](https://github.com/ShaMan123/gha-form-dropdown-options/actions/workflows/update_bug_report.yml/badge.svg)](https://github.com/ShaMan123/gha-form-dropdown-options/actions/workflows/update_bug_report.yml)

A github action populating options for an [issue forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms) dropdown.

## Looking for a _Version_ dropdown?

Checkout [![issue-forms-version-dropdown](https://img.shields.io/github/v/tag/ShaMan123/gha-populate-form-version?label=ShaMan123%2Fgha-populate-form-version%40&sort=semver)](https://github.com/marketplace/actions/issue-forms-version-dropdown)

## Configuring

- Follow [this workflow](.github/workflows/update_bug_report.yml).
- Replace the `uses: ./` directive to point to [![published action](https://img.shields.io/github/v/tag/ShaMan123/gha-form-dropdown-options?label=ShaMan123%2Fgha-form-dropdown-options%40&sort=semver)](https://github.com/marketplace/actions/issue-forms-dropdown-options).
- Refer to the `inputs` and `outputs` definitions in the [spec](action.yml).

## Live 🚀

This repo uses the action it defines.

- Take a look at the dropdowns populated into the [issue template](../../issues/new?template=bug_report.yml) using [this workflow](.github/workflows/update_bug_report.yml).
- Take a closer look at the [template file](./.github/template_report.yml) used to generate [`long_report.yml`](./.github/ISSUE_TEMPLATE/long_report.yml), see [Templates](#templates).
- Don't forget to checkout [#5](../../pull/5). Refer to [Creating a PR](#creating-a-pr) for more information.
- And you can view the [bot](../../commits?author=github-actions%5Bbot%5D) in action.

## Advanced Usage

### Templates

The `template` option introduces a _build_ step, sort of speaking.
It reads the `template` and outputs a generated file using `inputs.form`.
This means that you shouldn't edit the built form, as you wouldn't edit a build file.

The action will use `template` to build the form as follows:

- Parse `template`
- For each dropdown:
  - Determine if dropdown is static using `inputs.strategy`.
  - Static dropdowns (dropdowns with populated options in the template) take precedence over their built counterpart.\
    See an example [commit](https://github.com/ShaMan123/gha-form-dropdown-options/pull/2/commits/7cbd904caccb60c9bf52f066d11b303e439fe598).
  - Dynamic dropdowns are persisted from the built file in order to enable updating parts of a form from multiple runs.
- Set `inputs.options`
- Write `form`

Using a `template` is **suggested** for the following:

- Preserving comments: Parsing the yaml file will loose all comments
- DX: Maintaining forms with long dropdown options
- [Dynamic Substitution](#dynamic-substitution)

### Dynamic Substitution

- `{...}` : will populate the value from `template` if existing
- `{{...}}` : will populate the previous value from `form` if existing.

<details><summary>Dynamic Substitution Flow</summary>

**template.yml**

```yaml template.yml
    ...
    - type: dropdown
      id: _dropdown
      description: 'template says: {...}, build says: ...'
      options:
        - a
    ...
```

**workflow.yml**

```yaml workflow.yml
    ...
    with:
      id: _dropdown
      description: 'template says: {...}, build says: ...'
      options: {...}, b
    ...
```

**build.yml #1**

```yaml build.yml #1
    ...
    with:
      id: _dropdown
      description: 'template says: template says: {...}, build says: ..., build says: ...'
      options:
        - a
        - b
    ...
```

**workflow.yml**

```yaml workflow.yml
    ...
    with:
      id: _dropdown
      description: 'template says: {{...}}, build says: ...'
      options: {{...}}, c
    ...
```

**build.yml #2**

```yaml build.yml #2
    ...
    with:
      id: _dropdown
      description: 'template says: template says: template says: {...}, build says: ..., build says: ..., build says: ...'
      options:
        - a
        - b
        - c
    ...
```

**workflow.yml**

```yaml workflow.yml
    ...
    with:
      id: _dropdown
      description: 'template says: {{...}} but build says: ...'
      options: {...}, d
    ...
```

**build.yml #3**

```yaml build.yml #3
    ...
    with:
      id: _dropdown
      description: 'template says: template says: template says: template says: {...}, build says: ..., build says: ..., build says: ... but build says: ...'
      options:
        - a
        - d
    ...
```

</details>

Take a look at the [template file](./.github/template_report.yml) used to generate the [built file](./.github/ISSUE_TEMPLATE/long_report.yml) for a live example.

### Conflicting Runs

Since workflows run in concurrency you may encounter a case in which a number of runs are trying to modify and commit the same file.\
This might result in a merge conflict.\
If that is the case the action will fail.

_Consider the following:_\
The labels of this repo are populated into a dropdown (see [Live](#live)).\
Modifying a label more than once in a short period of time (before the previous runs completed) will fail to update the form.

Consider handling failures in a consequent step or use the `dry_run` option to prevent the action from trying to commit in the first place and handle that yourself.

### Creating a PR

Consider using [![create-pull-request](https://img.shields.io/github/v/release/peter-evans/create-pull-request?label=peter-evans%2Fcreate-pull-request&sort=semver)](https://github.com/marketplace/actions/create-pull-request) in order to commit changes in an orderly and safe fashion to a generated PR.

Refer to the `update-long-report` job in [this workflow](.github/workflows/update_bug_report.yml) to see a usage example and to [PR #5](../../pull/5).\
Don't forget to use the `dry_run` option.

## Developing

```bash
npm i
npm start
npm test
```

## Release

Create a release after bumping a version and committing it:

```
npm version <major/minor/patch>
```
