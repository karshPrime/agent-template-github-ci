# Changelog

## Added

* 3.0.1
   * the changelog lists changes under the version they were released in, newest first
* 3.0.0
   * VERSION holds a plain X.Y.Z and CI adds the v for release tags
   * existing projects remove the v from VERSION once
* 2.0.0
   * CI runs make format-dry-run, build, test and release from the project's Makefile, replacing run.sh
* 1.1.1
   * lint passes again now that CI runs on Ubuntu 26.04, a runner the pinned actionlint does not know yet
* 1.1.0
   * shared Renovate settings that templates and projects extend
   * non-major updates grouped into one weekly pull request with a reminder to bump VERSION
   * new projects ship a renovate.json that uses them
* 1.0.1
   * lint passes in CI again, and runs a pinned shellcheck so it matches a local run
* 1.0.0
   * shared GitHub CI for project templates, checking format, build, tests and release builds in the dev container
   * a VERSION bump required on every branch
   * a GitHub release with the merged pull request's description on each merge to main
