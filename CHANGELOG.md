# Changelog

## added/modified:

* feature/initial-template: shared GitHub CI for project templates, checking format, build, tests and release builds in the dev container, requiring a VERSION bump on every branch, and publishing a GitHub release with the merged pull request's description on each merge to main.
* fix/lint-shellcheck: lint passes in CI again, and runs a pinned shellcheck so it matches a local run.
* feature/shared-renovate: shared Renovate settings that templates and projects extend, grouping non-major updates into one weekly pull request with a reminder to bump VERSION; new projects ship a renovate.json that uses them.
* fix/ubuntu-26-ci: lint passes again now that CI runs on Ubuntu 26.04, a runner the pinned actionlint does not know yet.
* feature/makefile: CI runs make format-dry-run, build, test and release from the project's Makefile, replacing run.sh.
