# Bumping Major Version Number

This document outlines the process for bumping the major version of Chef Infra Client for the yearly release.

## Preparing Ohai

Chef consumes Ohai from GitHub as both a runtime dependency and a testing dependency for Test Kitchen validations in Buildkite. Ohai's version is tightly coupled to that of Chef Infra Client so the first step in bumping the major release in the chef/chef repo is to bump the major version of Ohai.

### Create a new stable branch of Ohai

On your local machine fork the current master branch to a new stable branch. For example: `git checkout -b 15-stable`. You’ll then want to edit the Expeditor config.yml file to update the branch configs like this:

https://github.com/chef/ohai/commit/ad208165619425dd7886b2de3f168b49ded25146

With the expeditor config complete push the branch `git push --set-upstream origin 15-stable`

### Bump Ohai master to the new major version

Create a PR which:

- Edits the `VERSION` file in the root of the repository to the new major release
- Updates the `chef-config` dependency to allow for the new major release of Chef Infra in `ohai.gemspec`

## Fork Chef master to a stable branch

Before bumping the major version of Chef Infra we want to fork off the current master to a new stable branch, which will be used to build hotfix releases. We support the N-1 version of Chef Infra Client for a year after the release of a new major version. For example Chef Infra Client 16 was released in April 2020, at which point Chef Infra Client 15 became the N-1 release. Chef Infra Client 15 will then be maintained with critical bug and security fixes until April 2021.

On your local machine fork the current master branch to a new stable branch. For example: `git checkout -b chef-15` and `git push --set-upstream origin chef-15`, which can then be pushed up to GitHub.

### Create a branch to fixup your new stable branch for release

Once you’ve forked to a new stable branch such as `chef-15` you’ll want to create a new branch so you can build a PR, which will get this branch ready for release:

- In ./expeditor/config.yml add the version_constraint for the new branch, update the version_constraint for master to match the new major version, and remove all the update_dep.sh subscriptions which don’t work against stable branches.
- In readme.md update the buildkite badge to point to the new stable branch image and link instead of pointing to master.
- In kitchen-tests/Gemfile update the Ohai branch to point to the new Ohai stable
- In kitchen-tests/kitchen.yml update chef_version to be your new stable version and not current. Ex: 15
- In tasks/bin/run_external_test update the ohai branch to point to your new stable ohai branch
- In Gemfile set ohai to pull from the ohai stable branch
- Run `rake dependencies:update`

Example PR for Chef 15: https://github.com/chef/chef/pull/9236

## Bump master for the new major release

Create a PR that performs the following:

- Update the version in the VERSION file
- Update chef.gemspec to point to the new ohai major release
- Update .expeditor/config.yml to include the new version_constraints you setup on your stable branch and an updated project alias
- run `rake dependencies:update`
