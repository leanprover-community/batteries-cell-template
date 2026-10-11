# Batteries Cell Template

A template for creating a Batteries cell: an independent Lean 4 repository that can be safely
imported together with [Batteries](https://github.com/leanprover-community/batteries). Cells are
kept up to date with Batteries and compatible with each other by the Batteries maintainers. See
the "Batteries Cell Network" section of the Batteries README for the current list of cells.

## Creating a cell

1. Click "Use this template" and turn on the "Include all branches" option, so that the new
   repository gets both the `main` and `nightly-testing` branches.

2. [Create a fine-grained personal access token][pat] with access to the new repository and with
   read and write permissions for contents, pull requests and workflows. Save it as a repository
   secret named `BATTERIES_CELL_TOKEN`. The workflows push and open pull requests with this
   token; pushes made with the default `GITHUB_TOKEN` would not trigger the CI and CD workflows.

3. Rename the package and library on both branches:
   - in `lakefile.toml`, replace the package name `batteries-cell-test` and the library
     `BatteriesCell` with your own names, and update `defaultTargets` to match;
   - rename `BatteriesCell.lean` to the new library's root module.

   Keep the `batteries` requirement: `rev = "main"` on `main` and `rev = "nightly-testing"` on
   `nightly-testing`.

4. Replace this README with one describing your cell, including how to require it:

   ```toml
   [[require]]
   name = "your-cell"
   git = "https://github.com/your-org/your-cell"
   rev = "main"
   ```

Keep the GitHub workflows as they are: they are what keeps the cell in step with Batteries.

## Branches and workflows

The `main` branch builds against Batteries `main` and its Lean toolchain. The `nightly-testing`
branch builds against Batteries `nightly-testing` and the latest Lean nightly toolchain.

- **CI Pipeline** (`ci-pipeline.yml`) builds every pull request to `main` or `nightly-testing`.
- **CD Pipeline** (`cd-pipeline.yml`) builds every push to `main` or `nightly-testing`. After a
  successful build on `main`, it merges `main` into `nightly-testing`, keeping the
  `lean-toolchain`, `lakefile.toml` and `lake-manifest.json` of `nightly-testing`.
- **Update Nightly Toolchain** (`update-nightly-testing-toolchain.yml`) runs daily. When the
  toolchain of Batteries `nightly-testing` changes, it updates the toolchain and manifest of
  `nightly-testing` and pushes the result.
- **Update Main Toolchain** (`update-main-toolchain.yml`) runs daily. When the toolchain of
  Batteries `main` changes, it merges `nightly-testing` into a new branch, sets the toolchain to
  match Batteries `main`, keeps the `main` lakefile, and opens a pull request to `main`. Changes
  needed to adapt to a new toolchain can therefore be made on `nightly-testing` ahead of time.
- **Build** (`build.yml`) is the shared build job, run with `--wfail` so that warnings fail the
  build. Every default target is built, so include your test library in `defaultTargets` if you
  want CI to run your tests.

All workflows can also be started by hand from the Actions tab.

[pat]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token
