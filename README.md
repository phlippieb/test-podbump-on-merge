# test-podbump-on-merge
A dummy repo to workshop a workflow where we do pod bumps on merge

# The current problem

Currently on the Health pod, and presumably some others, the pod bump aspect of our workflow is as follows:

1. I do some feature work on a branch
2. I create a pod bump (in the podspec) for my work
3. I create a PR with my changes, including the pod bump

Under isolated conditions, this works fine and even allows us to see during review whether a pod bump is present and appropriate. But in reality, several people are often marshalling their PRs through the process simultaneously, resulting in the following more convuluted example scenario:

1. I do some feature work on a branch
2. I create a pod bump
3. I create a PR with my changes
4. Someone else merges their PR, which causes the pod version of the base branch to update
5. I create a new pod bump
6. The approvals on my PR are dismissed, and I need to re-request reviews

The dismissed reviews in step 6 almost act like a backoff mechanism which gives other PRs the opportunity to be merged ahead of mine, which can cause the same process to be repeated multiple times.

# The idea

In order to address this problem, I would like to move the pod bump step so that it is performed automatically after the PR is merged. I think this makes sense for these reasons:

- During the PR process, it is not important to review which specific version will result from the pod bump; rather, it is only important to verify that the correct *type* of bump (patch vs minor vs major) will be performed.
- Once you have approval for the PR, and in particular for the pod bump you want to perform, it doesn't make sense to lose that approval on the grounds that another unrelated PR was merged. If Github can auto-merge the base into your PR without affecting your code, the effect of merging your PR will remain the same, both in terms of the code change (as verified by the automated unit tests) and in terms of the version bump (since it is defined by type).
- The pod bump is associated with a set of code changes made to the base. This relationship remains intact.

# Goals

The following outcomes would be good, if we can achieve them:

- Instead of having to perform a pod bump on your PR, be able to specify the type of bump you want
- Use CI to require you to specify the type of bump, so that we don't accidentally miss this
- Automatically perform the bump when the PR is merged
 
# Implementation

I'd like to leverage Github Actions to implement this. I know that some of the other teams have already implemented reusable actions that can be used to perform the actual pod bump. On our side, it could then work as follows:

- When you open a PR, you are required to set a label to specify the type of bump you need
  - Your options are patch, minor, major, or none (specified explicitly)
  - We might add a Github Actions check to ensure that the PR cannot be merged into master without supplying such a label
- Reviewers will be able to see which bump you intend to perform, and approve or request changes based on that
- When you merge, GitHub Actions will run a specific set of actions (based on the push event on master):
  - Check for labels associated with the merged branch; this might be possible using [this Github API](https://stackoverflow.com/questions/37345224/github-get-labels-of-pull-request-from-api)
  - Based on the label, decide what kind of bump to perform
  - Use the existing actions to perform and commit the bump
- As an added bonus, we could then kick off a `manual_pod_release` action on Bitrise

# Steps to do

This repo will act as a playground to implement a representative solution. The steps are as follows:

- [x] Create a dummy file that can be changed to simulate PR changes (DONE: See file.swift)
- [x] (https://github.com/phlippieb-discovery/test-podbump-on-merge/issues/1) Create labels for the types of version bumps
- [x] (https://github.com/phlippieb-discovery/test-podbump-on-merge/issues/2) Implement the requirement that a type of version bump must be specified in order for a PR to be merged
- [x] (https://github.com/phlippieb-discovery/test-podbump-on-merge/issues/3) Create the "on-merge" action
- [x] (https://github.com/phlippieb-discovery/test-podbump-on-merge/issues/4) On merge, determine which type of pod bump is required based on the label of the last-merged PR
