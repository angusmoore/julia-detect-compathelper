# julia-detect-dependabot Action

**Please note**: This action is not registered in the marketplace and is experimental. You can still use it in your own
workflows by referencing it as `angusmoore/julia-detect-dependabot@main`. But please treat as experimental for now.

This action checks if the current branch that the action is running on is a CompatHelper or dependabot
branch. If so, it sets an output variable `is_dependabot` that you can check in subsequent steps.

the [julia-actions/julia-runtest](https://github.com/julia-actions/julia-runtest) action does this automatically
when running tests. If you are using julia-runtest, you *do not* need this action.

This action is helpful if you are manually invoking `Pkg.test()` but want to be able to easily set
`force_latest_compatible_version` on CompatHelper pull requests (as is now standard for julia-runtest. This
situation might arise because you need to pass in custom arguments, or flags to the julia process that are not
supported by julia-runtest.

See also these issues
[1](https://github.com/julia-actions/julia-runtest/issues/47)
[2](https://github.com/JuliaRegistries/CompatHelper.jl/issues/298)
for discussion of automatically setting `force_latest_compatible_version` automatically in CI scrits, which this action
tries to solve (specifically this is row 2 in [1](https://github.com/julia-actions/julia-runtest/issues/47)).

## Usage

Julia needs to be installed before this action can run. This can easily be achieved with the [setup-julia](https://github.com/marketplace/actions/setup-julia-environment) action.

You need to provide an `id` for the action, so that you can access the output variable `is_dependabot` in subsequent
github actions steps.

You can access the result `is_dependabot` (which is a *string* of either "true" or "false") using the github
actions [steps context](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context). Eg, if you
set the `id` for the `julia-detect-dependabot` action as `dependabot`, you would access the value in subsequent steps
as: ${{ steps.dependabot.outputs.is_dependabot }}

An example workflow that uses this action might look like this:

```yaml
name: Run tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        julia-version: ['1', 'nightly']
        julia-arch: [x64, x86]
        os: [ubuntu-latest]

    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@v1
        with:
          version: ${{ matrix.julia-version }}
          arch: ${{ matrix.julia-arch }}
      - uses: angusmoore/julia-detect-dependabot@main
        id: dependabot
      - name: Custom package tests
        run: |
            julia --project -e 'import Pkg; Pkg.test(; force_latest_compatible_version = parse(Bool, ENV["IS_DEPENDABOT"]))'
        env:
          is_dependabot: ${{ steps.dependabot.outputs.is_dependabot }}

```
