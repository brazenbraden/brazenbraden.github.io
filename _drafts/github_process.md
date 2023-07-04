The following is a guide for improving workflow, clarity of communication, safety of releases and optimisation of developer, product and QA testing time.

This is an evolving document.

## PRs (Pull Requests)
### GitHub

Our GitHub PRs follow a few conventions, the details of which can be seen here: [Git Conventions](url_here)

Once a PR is ready for review, it will require:

- A short but clear description, useful when searching through old PRs
- A link to the more detailed ticket (GitHub Issue / Basecamp / Other project management tool) with the complete details needed for QA
- The owner(s) be assigned to the PR
- Link other relevant PRs to connect GitHub timelines

### GitHub Issue / Basecamp Ticket

The ticket we create for our PR is where the majority of information regarding the code changes will go. In order for clear communication between developers, product and QA, the following sections (including but not limited to) will be filled out as best as possible where required:

- Description - A more complete description of the PR. Why was the change needed? What does the change do and how does it solve the problem?

- Links - Links to all PRs required to complete the changes detailed in the description. If there are multiple PRs all related, they should all be defined here in a list prefixed by repo so when it comes to approving and merging the changes, QA can easily find all required PRs across repos.

Any additional links should be included here, for example:

> Zendesks
> Rollbars
> Other Issues / Tickets
> External Resources

- Integration Environment - Integration environment details allowing other developers and QA to quickly find the CI\CD job where the environment was deployed, as well as the URL and SSH ip.

- Testing Notes - Any notes QA need to know before beginning the manual test process as well as possible reproduction steps to perform in order to test the code change.

- Migrations - (Optional) Inform product and QA that the PR includes migrations and a release will have to be created after merge.

See these example tickets:
bug: Tag priorities not set on create
feature: Re-fetch transcripts for media from BabelFish

Once A PR is ready for testing, given the conditions in the section below, QA and Product can be added to the **"Assigned to"** section so they are notified and the ticket is easily discoverable.

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

## Notes
* It may happen that a PR requires an accompanying PR from another branch to complete the changes which could result in the e2e tests failing as CircleCI builds master branch on all other repositories. If this is the case, include a note about that in the issue description.