---
layout: post
title: Git Conventions (part 1/3)
crawlertitle: Git branch and commit conventions
summary: Evolving git branching and commit conventions
date:   2023-07-04
categories: posts
tags: ['git conventions']
bg: "post-git-conventions.jpg"
---

Over the last decade or so, I have had the opportunity to experiment with various git usage strategies and styles. Ranging from branch names, to pull request descriptions, a lot of time has been spent thinking about how to best describe our work to convey as much information clearly and concisely to others. There are a wealth of strategies or style guides on how to best utilise the git ecosystem but there is no silver bullet that solves all potential scenarios. What follows is an outline of some of the rules and guides I have incorporated into my daily git life which has best worked for my team and me.

This post is more evolving personal documentation than directed advice but there might be a nugget or two worth taking from it.

## Branches
99% of the time, code changes fall into one of a handful of categories/types. When creating a new branch, naming it something clear and meaningful can help other developers know at a glance what sort of changes are been introduced. It can also simplify finding a branch using tab completion locally when jumping between branches.

### Branch Types
The following branch **types** should suit most use cases. These can be refined or altered depending on your use case (for instance, one might not utilise the **style** type if they work on CLI scripts for a living)

- **feature** _- a new feature_
- **fix** _- bug fixes, hotfixes, typos, etc_
- **style** _- UI / UX style changes_
- **test** _- adding or refactoring tests; no production code changes_
- **housekeeping** _- everything from code refactoring to updating documentation_
- **devops** _- updates to configuration files, software versions, etc_

### Branch Format

The proposed branch format will make it easy to see at a glance what type of work has been done in the branch with a clean and easy-to-read branch name. The max length of a branch name is 255 characters but where possible, I try to keep the length to no more than 50 characters for readability purposes.

```
type-name_of_branch
```

The type is separated from the branch name with a `-` while the rest of the name is done in snake case (words separated by `_`)

**Examples branch names:**
```
feature-add_user_authentication
fix-about_page_infinite_redirect
test-user_name_validation
style-switch_from_bootstrap_to_material_ui
housekeeping-extract_auth_logic_into_service_object
devops-upgrade_rails_to_2.6.5
housekeeping-update_onboarding_docs
```

## Commits
Developing a structured format for our commit messages will clarify each commit's purpose resulting in a cleaner git history. At a glance, one will know if a commit is intended to add a feature, fix a bug or just improve code quality. We should aim to keep the work done within a branch small and testable, often resulting in there only being a single commit per branch, however, it's better to have lots of small atomic commits within a branch than having all the commits squashed into one before merging. This makes bisecting introduced bugs a lot easier.

### Commit Message Template:

This is a sample commit template I have been using for a while now. As I try to keep my commits granular and bespoke, I define the first line leaving everything else commented out. However, if you are pushing up a commit that involves a lot of changes, perhaps references a specific issue or task item, or includes changes that could affect future development, it is recommended to flesh out the commit message with as much information as possible to help future you (or team members) get a clear picture of the work done by looking at your commit alone.

The commit subject, as seen below, has the convention of starting with the type of work being committed (bug, feature, housekeeping, etc) all in lowercase, followed by a semicolon, space and commit message, starting capialised, in the imperative form, without any full stop at the end.

```
type: Summary of change

# **--- Proposed Changes ---**
#
#
# Issue / Task: [name](url)

# --- COMMIT END --- (informational only, do not uncomment)
# <type>: <subject> (Approx 50 characters)
# |<----  Using a Maximum Of 50 Characters  ---->|
# Explain why this change is being made
# |<----   Try To Limit Each Line to a Maximum Of 72 Characters   ---->|
# --------------------
# Type can be
#    feature      (new feature)
#    fix          (bug fix)
#    style        (UI / UX style changes)
#    test         (adding or refactoring tests; no production code change)
#    housekeeping (refactoring code or adding documentation)
#    devops       (background / architecture changes)
# --------------------
# Remember to
#   - Use the imperative mood in the subject line
#   - Do not end the subject line with a period
#   - Separate subject from body with a blank line
#   - Use the body to explain what and why vs. how
# --------------------
```

All the lines starting with a `#` are commented out and will not be displayed when viewing the commit message so if you intend to add future detail to your commit message, be sure to uncomment the lines necessary.

You can set up your default git commit message template with
```
git config --global commit.template ~/.gitmessage
```

**Example commit messages:**
```
feature: Add user authentication
fix: About page infinite redirect
style: Add new icon pack
test: User name validation
housekeeping: Update onboarding docs
devops: Upgrade rails to 2.6.5
```

Very often when jumping into a piece of work, I might not know right off the bat what the commit message will be as it tends to evolve as the work gets completed. In this case, I usually start my commit with something like

```
wip: Not sure what the issue is
```
and later on, once having completed the work and getting the branch ready for review, rename the commit messages appropriately during an interactive rebase (`git rebase -i`).

## Closing Comments

This is by no means the definitive git convention standard to employ and it is something that is continually evolving and morphing as I find myself working on new and different projects. Everyone's experience is different and must be dealt with accordingly but, perhaps there are a few nuggets here that could help you guide a more structured and concise git workflow.

To further the conventions, I have also put together standard workflows, processes, and templates when it comes to GitHub Pull Requests (PR) and Issues affecting code reviews, QA testing, and deployment, but that is a story for another post.

## Additional Resources

1.) A lot of the inspiration for these conventions has been taken from "Conventional Commits".

[https://www.conventionalcommits.org/en/v1.0.0/#why-use-conventional-commits](https://www.conventionalcommits.org/en/v1.0.0/#why-use-conventional-commits)
[https://marcodenisi.dev/en/blog/why-you-should-use-conventional-commits/](https://marcodenisi.dev/en/blog/why-you-should-use-conventional-commits/)

2.) A classic article on how best to write commit messages. I have gleaned a lot of the commit message advice in this article to supplement my variation.

[https://cbea.ms/git-commit/#imperative](https://cbea.ms/git-commit/#imperative)

3.) A commit linting tool you can incorporate into your workflow to enforce a standard of commit messages. Can be configured in a pre-commit GIT hook.

[https://commitlint.js.org/#/](https://commitlint.js.org/#/)
