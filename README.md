git-conflict-separate-files
===========================

Introduction
------------

Within my current project, I'm working with a team of collegues
on software services. Quite often, we manipulate the same
service in parallel meaning I ("Uli") implement feature U
within the service whereas my collegue ("Tom") implements
feature T. We use separate feature branches within
git for this, so we can implement our features independently
from each other.

Sometimes, we work on the same files in parallel.
This typically leads merge conflicts later on.

I personally don't like bi-directional merge flows
within "main" and my feature branch. So typically, I rebase
my feature branch onto the top of the "main" branch from
time to time.

Quite often, the rebase raises merge conflicts.
Typically, they are easy to solve but sometimes, this
is tedious. This is especially true for "rebase" (opposed to "merge"),
since the merge conflict may be raised for many changes.

Sometimes you think: It was a really bad idea to integrate
my new unit tests into the existing unit test class since
this class now leads to most of the conflicts. I wish I
would have created a separate unit test class right from
the start.

I found a way to fix this later on!

Setting Up This Repo
--------------------

### Groundwork

```
WORK=(path-to-location-where-you-store-your-work)
OLD_PWD="$(pwd)"
cd $WORK

# Create local repo
git init git-conflict-separate-files
cd git-conflict-separate-file

# Add an initial commit (I like using an empty commit for this)
git commit --allow-empty -m "Initial commit"

# Create this file (README.md)
git add README.md
git commit -m "Added a first version of README.md"

# Create project on github
# Determine project url: git@github.com:uli-heller/git-conflict-separate-files.git
# "Link" the local repo to the github project
git remote add origin git@github.com:uli-heller/git-conflict-separate-files.git
git push -u origin main

cd "${OLD_PWD}"
```
