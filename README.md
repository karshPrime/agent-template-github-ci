# GitHub CI Template

[![Copier](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/copier-org/copier/master/img/badge/badge-grayscale-inverted-border-orange.json)](https://github.com/copier-org/copier)

This repository provides the GitHub Actions workflow shared by every project template. It is copied into each language template, so every project generated from one ships with the same CI, and updates to the workflow reach every template the same way.


### Prerequisites

Install the Copier command:
* **Quick option:** `uv tool install copier`
* **Documentation:** https://copier.readthedocs.io/en/stable


### Add to a Template

From the root of a language template repository, copy into its `template/` directory:

```bash
$ copier copy gh:karshosphere/template-github-ci template
```
* There are no questions, and no `--trust` is needed as the template runs no commands.
* The answers file is written to `template/.github/.copier-ci-answers.yml`.

To bring in a newer release of the workflow later:

```bash
$ copier update -a .github/.copier-ci-answers.yml template
```


### What a Project Gets

* `.github/workflows/ci.yml`: the shared workflow.
* `.github/.copier-ci-answers.yml`: the record of which release of this template the workflow came from.
* `renovate.json`: extends the shared Renovate settings below. The project owns this file too, so it can add its own rules underneath.


### What a Project Must Provide

* A `Makefile` at the project root, with these targets:
  * `format-dry-run`: check formatting without changing files.
  * `build`: build the project.
  * `test`: run the tests.
  * `release`: build every release target the project ships, writing the files to attach to the GitHub release directly into `dist/`. A project with no release targets leaves `dist/` empty.
  * Each fails on error. Longer actions can live in `scripts/` and be called from the Makefile.
  * CI passes the version being released to every target as `VERSION=X.Y.Z`. Anything that reports the version should use it, falling back to `git describe --tags` for local builds.
* `.devcontainer/devcontainer.json`, with `make` installed: CI runs every target inside it, so the toolchain is the one the project already defines.


### The Workflow

* **version:** on every pull request and push to `main`, works out the next version from the last `vX.Y.Z` tag and the commits since it, and shows it as a notice on the run. A pull request shows the version merging it would release.
* **checks:** starts the dev container, then runs `make format-dry-run`, `make build`, `make test` and `make release` inside it, each as its own step and each given `VERSION=X.Y.Z`. Running `make release` on pull requests means a broken release build fails before merge.
* **release:** on push to `main`, once both jobs pass, publishes a GitHub release named `v` plus the version, such as `v1.2.3`, which also creates the tag. Its notes are the description of the pull request that was merged, and every file `make release` left directly in `dist/` is attached. The release is only published once every file is attached.

The version comes from the commit messages, so there is no file to bump:
* `X` when any commit is marked breaking, with `!` after its type, such as `feat!:`, or a `BREAKING CHANGE:` line in its body.
* `Y` when any commit starts with `feat:`.
* `Z` for everything else, such as `fix:`, `chore:` and `docs:`, which includes Renovate's updates.
* The biggest one wins, and a project's first release is `0.1.0`.

Merges to `main` are released one at a time, in order. If several land while a run is queued, the next release takes in all of them, with each pull request's description under its own heading. A squash merge is judged by the squashed commit's title alone.

Projects generated before 4.0.0 hold their version in `VERSION`. Updating to 4.0.0 removes it, and the existing tags carry on as before.

The workflow is read-only apart from the release job, which needs write access to publish the release and read access to pull requests for its notes. It runs on `pull_request`, never `pull_request_target`, so code from a fork never runs with the repository's credentials. Actions are pinned to commit SHAs, and Node and the Dev Container CLI to exact versions.


### Shared Renovate Settings

`default.json` at the root of this repository holds the Renovate settings shared by every template and project. Any repository uses them with:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>karshosphere/template-github-ci"]
}
```
* Renovate only opens or updates pull requests before 6am on Sunday Melbourne time, security updates included, once a release is at least 7 days old.
* Every update, majors too, arrives in one grouped pull request.
* Renovate's commits are `chore:`, so merging its pull request releases a `Z` bump with no other change.
* Versions marked with a `# renovate:` comment in workflows and Dockerfiles are kept up to date too.

Changes to `default.json` reach every repository on Renovate's next run once merged, with no `copier update` needed.


### This Repository's CI

`.github/workflows/ci.yml` at the root checks this repository itself, not the projects: the same version from its commits and a GitHub release on merge, plus `actionlint` over both the shipped workflow and its own, using a pinned shellcheck rather than the runner's preinstalled one so results match a local run. Runner labels newer than the pinned actionlint are declared in `.github/actionlint.yaml`. Renovate, through the shared settings, proposes updates to the pinned versions in both.
