---
layout: post
title: Git Conventions
crawlertitle: Git branch and commit conventions
summary: Evolved Git branching and commit conventions
date:   2023-06-27
categories: posts
tags: ['git conventions']
bg: "post-git-conventions.jpg"
---

Over the last decade or so, I have had the opportunity to experiment with various git usage strategies and styles. Ranging from branch names, to pull request descriptions, a lot of time has been spent thinking about how to best describe our work in order to convey as much information clearly and consisely to others. There are a wealth of strategies or style guides on how to best utilise the git ecosystem but there is no single silver bullet that solves all potential scenarios. What follows is an outline of some of the rules and guides I have incorporated into my daily git life which has best worked for my team and I. 

## Branches
99% of the time, code changes being fall into one of a handful of categories / types. When creating a new branch, naming it something clear and meaningful can help other developers know at a glance what sort of changes are been introduced. It can also simplify finding a branch using tab completion locally when jumping between branches.

### Branch Types
The following branch **types** should suite most use cases. Note that in my case, focusing primarily on backend Ruby development, I haven't had need to updating front end UI / UX and therefore there are probably a few types more suitable for those sorts of commits, like **`style`**.

- **feature** _- a new feature_
- **fix** _- bug fixes, hot fixes, typos, etc_
- **test** _- adding or refactoring tests; no production code changes_
- **housekeeping** _- everything from code refactoring to updating documentation_
- **devops** _- updates to configuration files, software versions, etc_

### Branch Format

The proposed branch format will make it easy to see at a glance what type of work has been done in the branch with a clean and easy to read branch name. The max length of a branch name is 255 characters but where possible, try to keep the length to no more than 50 characters.
```
type-name_of_branch
```

The type is seperated from the branch name with a `-` while the rest of the name is done in snake case (words seperated by `_`)

**Examples branch names:**
```
feature-adds_user_authentication
fix-about_page_infinite_redirect
test-user_name_validation
feature-create_client_dashboard
housekeeping-extract_auth_logic_into_service_object
devops-upgrade_rails_to_2.6.5
housekeeping-updates_onboarding_docs
```

## Commits
By developing a structured format for our commit messages, it will clarify each commits purpose resulting in a cleaner git history. At a glance, one will know if a commits intention was to add a feature, fix a bug or just improve code quality. We should aim to keep the work done within a branch small and testable, often resulting in there only being a single commit, however, its better to have lots of small standalone commits within a branch than having all the commits squashed into one before merging.

This commit message template specifies the structure of the commit "title / headline" and is the desired format for all git commits. I have included a section starting 

### Commit Message Template:

You can set up your default git commit message template with
```
git config --global commit.template ~/.gitmessage
```

This is a sample commit template I have been using for a while now. As I try keep my commits granular and bespoke, I define the first line leaving everything else commented out. However, if you are pushing up a commit that involves a lot of changes, perhaps references a specific issue or task item, includes changes which could affect future development, it is recommended to flesh out the commit message with as much information as possible so as to help future you (or team members) get a clear picture of the work done by looking at your commit alone.

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

**Example commit messages:**
```
feature: Adds user authentication
fix: About page infinite redirect
test: User name validation
housekeeping: Updates onboarding docs
devops: Upgrade rails to 2.6.5
```

Very often when jumping into a piece of work, I might not know right off the bat what the commit message will be as it tends to evolve over time as the work gets completed. In this case, I usually start my commit with something like
```
wip: Not sure what the issue is
```
and later on, once having completed the work and getting the branch ready for review, rename the commit messages appropriately during an interactive rebase (`git rebase -i`).

## Closing Comments

This is by no means the definitive git convention standard to employ and it something which is continually evolving and morphing as I find myself working on new and different projects. Everyone's experience is different and must be dealt with accordingly but, perhaps there are a few nuggets here that could help you guide a more structured and concise git workflow. 

To further the conventions, I have also put together standard workflows, processes and templates when it comes to Github Pull Requests (PR) and Issues affecting code reviews, QA testing and deployment, but that is a story for another post.

## Additional Resources

1.) A lot of the inspiration for these conventions have been taken from "Conventional Commits"

[https://www.conventionalcommits.org/en/v1.0.0/#why-use-conventional-commits](https://www.conventionalcommits.org/en/v1.0.0/#why-use-conventional-commits)
[https://marcodenisi.dev/en/blog/why-you-should-use-conventional-commits/](https://marcodenisi.dev/en/blog/why-you-should-use-conventional-commits/)

2.) A commit linting tool you can incorporate into your workflow to enforce a standard of commit messages. Can be configured in a pre-commit GIT hook.

[https://commitlint.js.org/#/](https://commitlint.js.org/#/)