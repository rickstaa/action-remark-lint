# Action-remark-lint GitHub Action

[![Test](https://github.com/rickstaa/action-remark-lint/workflows/Test/badge.svg)](https://github.com/rickstaa/action-remark-lint/actions?query=workflow%3ATest)
[![release](https://github.com/rickstaa/action-remark-lint/workflows/release/badge.svg)](https://github.com/rickstaa/action-remark-lint/actions?query=workflow%3Arelease)
[![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/rickstaa/action-remark-lint?logo=github\&sort=semver)](https://github.com/rickstaa/action-remark-lint/releases)
[![action-bumpr supported](https://img.shields.io/badge/bumpr-supported-ff69b4?logo=github\&link=https://github.com/haya14busa/action-bumpr)](https://github.com/haya14busa/action-bumpr)

This action runs the [remark-lint linter](https://github.com/remarkjs/remark-lint) to check/format your markdown code on a push or pull request.

## Quickstart

```yml
name: remark-lint-action
on: [push, pull_request]
jobs:
  remark-lint:
    name: runner / remark-lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: remark-lint
        uses: rickstaa/action-remark-lint@v1
```

## Inputs

### `remark_args`

**optional**: Remark-lint input arguments. Defaults to `. --frail --quiet`.

### `fail_on_error`

**optional**: Exit code when remark-lint linting errors are found \[true, false]. Defaults to 'true'.

## Outputs

### `is_formatted`

Boolean specifying whether any files were formatted using the remark-lint linter.

## Advanced use cases

### Annotate changes

This action can be combined with [reviewdog/action-suggester](https://github.com/reviewdog/action-suggester) also to annotate any possible changes (uses `git diff`).

```yaml
name: remark-lint-action
on: [push, pull_request]
jobs:
  name: runner / remark-lint
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: Check files using the remark-lint linter
      uses: rickstaa/action-remark-lint@v1
      id: action_remark_lint
      with:
        remark_args: ". -o"
    - name: Annotate diff changes using reviewdog
      if: steps.action_remark_lint.outputs.is_formatted == 'true'
      uses: reviewdog/action-suggester@v1
      with:
        tool_name: remarkfmt
```

### Commit changes or create a pull request

This action can be combined with [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request) to also apply the annotated changes to the repository.

```yaml
name: remark-lint-action
on: [push, pull_request]
jobs:
  name: runner / remark-lint
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: Check files using remark-lint linter
      uses: rickstaa/action-remark-lint@v1
      id: action_remark_lint
      with:
        remark_args: ". -o"
    - name: Create Pull Request
      if: steps.action_remark_lint.outputs.is_formatted == 'true'
      uses: peter-evans/create-pull-request@v3
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        title: "Format Python code with remark-lint linter"
        commit-message: ":art: Format Python code with remark-lint"
        body: |
          There appear to be some python formatting errors in ${{ github.sha }}. This pull request
          uses the [remark-lint](https://github.com/remarkjs/remark-lint) linter to fix these issues.
        base: ${{ github.head_ref }} # Creates pull request onto pull request or commit branch
        branch: actions/remark-lint
```
