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
* `VERSION`: the project's version, starting at `v0.1.0`. The project owns this file; updates never overwrite it.
* `renovate.json`: extends the shared Renovate settings below. The project owns this file too, so it can add its own rules underneath.


### What a Project Must Provide

* `run.sh` at the project root, executable, accepting:
  * `--format-dry-run`: check formatting without changing files.
  * `--build`: build the project.
  * `--test`: run the tests.
  * `--release`: build every release target the project ships, writing the files to attach to the GitHub release directly into `dist/`. A project with no release targets leaves `dist/` empty.
  * Each exits non-zero on failure, and on any flag it does not know.
* `.devcontainer/devcontainer.json`: CI runs every `run.sh` check inside it, so the toolchain is the one the project already defines.


### The Workflow

* **version:** on every pull request and push to `main`, checks that `VERSION` is a single `vX.Y.Z` line, higher than every existing `vX.Y.Z` tag. A branch that does not bump `VERSION` fails here.
* **checks:** starts the dev container, then runs `./run.sh --format-dry-run`, `--build`, `--test` and `--release` inside it, each as its own step. Running `--release` on pull requests means a broken release build fails before merge.
* **release:** on push to `main`, once both jobs pass, publishes a GitHub release named after `VERSION`, which also creates the tag. Its notes are the description of the pull request that was merged, and every file `--release` left directly in `dist/` is attached. The release is only published once every file is attached.

Bump `VERSION` on every branch:
* `Z` for minor fixes.
* `Y` for minor features and chores.
* `X` for breaking changes.

The workflow is read-only apart from the release job, which needs write access to publish the release and read access to pull requests for its notes. It runs on `pull_request`, never `pull_request_target`, so code from a fork never runs with the repository's credentials. Actions are pinned to commit SHAs, and Node and the Dev Container CLI to exact versions.


### Shared Renovate Settings

`default.json` at the root of this repository holds the Renovate settings shared by every template and project. Any repository uses them with:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>karshosphere/template-github-ci"]
}
```
* Updates are proposed weekly, before 6am on Sunday Melbourne time, once a release is at least 7 days old.
* Minor, patch and digest updates arrive as one grouped pull request, and majors on their own.
* Renovate never touches `VERSION`, so bump it on the pull request before merging. Each pull request carries a reminder.
* Versions marked with a `# renovate:` comment in workflows and Dockerfiles are kept up to date too.

Changes to `default.json` reach every repository on Renovate's next run once merged, with no `copier update` needed.


### This Repository's CI

`.github/workflows/ci.yml` at the root checks this repository itself, not the projects: the same `VERSION` checks and tag on merge, plus `actionlint` over both the shipped workflow and its own, using a pinned shellcheck rather than the runner's preinstalled one so results match a local run. Renovate, through the shared settings, proposes updates to the pinned versions in both.
