---
layout: page
title: Development
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Contributors

If you'd like to contribute, please send a pull-request.
You can build the pull-request by creating a comment that matches exactly `/test` and nothing else.
This will trigger the mock and the integration tests. It takes a few moments to get the results.

Please provide unit tests and integration tests.

A project owner will be responding as soon as time permits.

# Project owners

Project owners can merge pull-requests.

## Pull-Requests

### Release drafter

In addition to the code review,
labels need to be applied to each pull-request to help the release drafter compute the next version number
and prepare the release notes for the upcoming release.

The next version number is computed based on the `version-resolver` labels configuration found in
[.github/.github/release-drafter.yml](https://github.com/jrestclients/.github/blob/main/.github/release-drafter.yml).

The release notes are taken from the git commit messages, and automatically link to issues.

### Merging pull-requests

The requirements for merging pull-requests are:

* Only project owners can merge pull-requests
* Pull-requests must be up to date with the main branch
* Pull-requests must pass the unit tests and integration tests github actions
* Pull-requests must be properly labelled

When the pull-request is merged, 
The release drafter workflow runs and updates the [next version release notes](https://github.com/jrestclients/jenkins-rest/releases) (only visible to owners).

## Releasing a new version

Project owners can release a new version.

To release a new version, edit the draft release notes.
Modify the release notes if needed, modify the next version number if needed.

When you click "Publish release", the `release.yml` workflow is executed.
This workflow runs the unit test and the integration tests.

If the test fails, the release process does not go further.

If the tests pass, the release is published to the staging repository in maven central.
Project owners can review the staging repository and release it 
(soon this will all be performed by the github action making it possible for all project owners to produce a new release).

If the publishing to maven central fails, the release process has failed, and clean up must be performed manually.
At this point, it is likely necessary to:

* identify and fix the problem via additional pull-requests
* inspect the internal project configuration directly in github if an internal error is suspected
* manually re-create the release notes (copy them!), as the github release action currently only triggers when the release is initially published.
* manually delete the staging repository in Maven Central
* manually remove the git tag that was created

