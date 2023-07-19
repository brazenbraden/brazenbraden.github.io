---
layout: post
title: Release Process
crawlertitle: Development release process
summary: Evolving process around QA, merging and releasing code changes
date:   2023-07-04
categories: posts
tags: ['release process']
bg: "post-release-process.jpg"
---

After having followed the various conventions defined by our [Git]() and [Github]() processes, we should now be in a good place to take the code that has been produced, get it ready for QA, merge and release it. Every organisation will have their own processes and standards set around this, some good, some bad. I worked with my team to build a robust and practicle release process which best suited our needs at the time.

As per the previous couple of posts, this is an evolving document which has been altered and tuned over the years. It may not apply to you and your situation but perhaps some of the suggestions here could help improve your workflow and overall developer happiness.

## QA / Testing
Before a PR undergoes manual testing, the following conditions will be checked:

Ensure all tests are green (unit and e2e) *

Have at least 2 approval reviews via the review process

Any comments on the PR that require resolving be resolved

Do a quick check yourself on your deployed integration environment

This ensures that the changes being tested have gone through the due process and reduce the chance that issues are discovered during manual testing. This could save resources (unnecessary integration environments), developer and QA time.

## Merging
Before a PR can be merged into master, it will be manually tested (in most cases - sometimes a PR may contain changes at the devops level which cannot be manually tested) and approved by QA. This will give the PR at least its 3rd green tick.

QA will do a quick code review to see if there are any changes to the packages.json file (in the platform repo) to ensure that no dev/alpha packages are included, that the packages have been updated to the final release build. If the packages need updating, the owner of the PR will be notified on the Basecamp ticket responsible for the PR.

For example, a PR with packages in the following state (i.e. with the "-alpha.x") should not be merged until the packages look like "1.10.64"

enter image description here

## Release
When it comes time to compile a release of changes, the collection of PRs that have been merged into master will be grouped under a release tag. Once the release has built and all the tests have passed, both unit and e2e, the release will be deployed.

In the event that a PR containing a database migration is merged, a release will be created straight away and deployed, to ensure the migration process proceeds smoothly. This is to ensure that there are no potential issues from compound migrations and also to allow quick and simple rollback should there be an issue.
