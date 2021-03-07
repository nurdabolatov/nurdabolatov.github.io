---
layout: post
title: How to change git history?
---

"As with any digital data, git history can also be rewritten", he thought when preparing his project for the open source community. He thought of multiple use cases where git history editor would have been useful:
- Changing the commit message convention. For example, when he decides to use only lowercase letters in all commit messages.
- Removing files with server configs when open-sourcing a web project.
- Switching to a new email address and deleting his embarrassing email address he came up with in high school.

Without further adieu, he quickly looked up how to remove his database credentials from git history.

These were the steps he performed to remove a file from git history:

1. Install [git-filter-repo](https://github.com/newren/git-filter-repo) 

   `brew install git-filter-repo`

2. Go to a repo folder

   `cd repo`

3. Remove `production.env` from all commits

   `git filter-repo --invert-paths --path production.env`

4. Hard push to the remote repo

   `git push -f`