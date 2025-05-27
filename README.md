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

See The Issue
-------------

### Extract The Repo

```sh
mkdir demo
xz -cd data/git-conflict-separate-files.tar.xz\
|(cd demo; tar xf -)
```

Note: It may be kind of irritating.
After extraction, you have these files and folders:

- git-conflict-separate-files
  - .git (probably hidden)
  - .gitignore (probably hidden)
  - README.md
  - MyService.md
  - MyServiceTest.md
  - data
    - git-conflict-separate-files.tar.xz
  - demo
    - git-conflict-separate-files
      - .git (probably hidden)
      - README.md
      - MyService.md
      - MyServiceTest.md

The demo works only underneath "demo/git-conflict-separate-files"!

### Change Your Working Directory

```
cd demo/git-conflict-separate-files
```

Make sure you're always underneath this folder!
Avoid doing anything underneath the checkout folder!

### Rebase "feature-u" to "main"

You realize that "feature-u" isn't based on HEAD of main:

```sh
$ git log --graph --oneline main feature-u
* a8504e8 (HEAD -> main) Update all branches
* 19798ea Added appendices
* 70b6393 Feature W
* c246429 More work on tests
* 18521ad Fixed test A
* 4c96670 Fixed feature V - pt2
* 7e009b0 Fixed feature V
* 8efc3e9 Typo
* 646c190 More docs related to branches
| * 2c8150f (feature-u) Work on U
| * abf7228 Work on U
| * cb8d160 U
| * fdf3406 Tests for U
| * 80d6cf3 Tests for U
| * 8023eb5 Tests for U
| * 1757663 Fix for V - tests
| * e57684f Fix for V
| * d123ed7 Started work on U
|/  
* 802dca2 Fixed test for A
* 4c6a178 Fixed junit test V, added test for A
* 399c664 Describe how to create branch feature
...
```

There are lots of changes in main which aren't in feature-u
(646c190..a8504e8). In order to simplify future merges from
feature-u to main, you decide to rebase it:

```sh
  # Make sure you have the latest branches
$ git checkout main
$ git pull
$ git checkout feature-u
$ git pull

  # Start the rebase
$ git rebase main
  # See lots of conflicts -> solve -> git rebase --continue
  # Tally: XXXX
$ git rebase --continue
```

Probably, you already hate this procedure!
You have to resolve lots of conflicts
and maybe you think: Why the hell
didn't I create a separate test file?

BTW: There are other workflows
where the number of conflicts is reduced.
For this demo, we **want** to use a workflow
based on "rebase"!

### Simulate Some Progress On Main And Rebase Again

```sh
git checkout main
git pull
git rebase main-2
# Now main has lots of additional changes

git checkout feature-u
git rebase main
# Lots of conflicts again -> solve -> git rebase --continue
git rebase --continue
```

### Merge Tom's Work On Main And Rebase Again

```sh
git checkout main
git merge feature-t
# Some conflicts -> solve -> git merge --continue
git merge continue
# Now main has  additional changes

git checkout feature-u
git rebase main
# Lots of conflicts again -> solve -> git rebase --continue
git rebase --continue
```

### Finally Merge Uli's Work

```
git checkout main
git merge feature-u
# No merge conflicts
```

Appendices
----------

### Setting Up This Repo

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

# Modify README.md, add several additional files, ...

# Start branch for Tom's feature
git tag feature-t-start
git checkout -b feature-t
git push --tags
git push -u origin feature-t

# Back to main
git checkout main
# Modify README.md -> commit, push
git commit -m "Describe how to create branch feature-t" .
git push
# Additional modifications...

# Start branch for Uli's feature
git tag feature-u-start
git checkout -b feature-u
git push --tags
git push -u origin feature-u

# Branch for future modifications of "main"
git checkout -b main-2
# Do some modifications
git commit -m "Work on ..."
git push -u origin main-2

# Back to main
git checkout main

cd "${OLD_PWD}"
```

### Using This Repo

```
WORK=(path-to-location-where-you-store-your-work)
OLD_PWD="$(pwd)"
cd $WORK

git clone git@github.com:uli-heller/git-conflict-separate-files.git

cd git-conflict-separate-files
git fetch --all

cd "${OLD_PWD}"
```

### Update All Branches

```sh
WORK=(path-to-location-where-you-store-your-work)
OLD_PWD="$(pwd)"
cd $WORK/git-conflict-separate-files

git branch -r\
|grep -v HEAD\
| cut -d / -f 2\
| while read -r b; do git checkout "$b"; git pull; done

cd "${OLD_PWD}"
```
