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

Move Tests Into A Separate File
-------------------------------

We've seen lots of conflicts related to "MyServiceTest.md".
We're trying to move our tests into a separate file "MyServiceFeatureUTest.md".
Hopefully, we will see much less conflicts!

### Extract The Repo Again

```sh
rm -rf demo
mkdir demo
xz -cd data/git-conflict-separate-files.tar.xz\
|(cd demo; tar xf -)
```

### Change Your Working Directory Again

```
cd demo/git-conflict-separate-files
```

Make sure you're always underneath this folder!
Avoid doing anything underneath the checkout folder!

### Find Common Ancestor For "main" And "feature-u"

```sh
$ git merge-base --all main feature-u
802dca213e80fdd45699e03dad8e7be4f99b2053

$ git log --graph --oneline main feature-u --not "$(git merge-base --all main feature-u)~"
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
```

### Rewrite Branch "feature-u" To Contain A Copy Of "MyServiceTest.md"

```sh
$ git checkout -b feature-u-2 802dca2
$ cp MyServiceTest.md MyServiceFeatureUTest.md
$ git add MyServiceFeatureUTest.md
$ git commit -m "Copied MyServiceTest.md -> MyServiceFeatureUTest.md"
[feature-u-2 b9ec52f] Copied MyServiceTest.md -> MyServiceFeatureUTest.md
 1 file changed, 51 insertions(+)
 create mode 100644 MyServiceFeatureUTest.md

$ git checkout feature-u
$ git rebase feature-u-2
Erfolgreich Rebase ausgeführt und refs/heads/feature-u aktualisiert.

$ git branch -d feature-u-2
Branch feature-u-2 entfernt (war b9ec52f).
```

### Review History

```sh
$ git merge-base --all main feature-u
802dca213e80fdd45699e03dad8e7be4f99b2053

$ git log --graph --oneline main feature-u --not "$(git merge-base --all main feature-u)~"
* ba43a18 (HEAD -> feature-u) Work on U
* 3e246ef Work on U
* 55087fe U
* a07c159 Tests for U
* bbba233 Tests for U
* 77e80cd Tests for U
* b85dc48 Fix for V - tests
* a7a20c2 Fix for V
* 35f6d75 Started work on U
* b9ec52f Copied MyServiceTest.md -> MyServiceFeatureUTest.md
| * a8504e8 (main) Update all branches
| * 19798ea Added appendices
| * 70b6393 Feature W
| * c246429 More work on tests
| * 18521ad Fixed test A
| * 4c96670 Fixed feature V - pt2
| * 7e009b0 Fixed feature V
| * 8efc3e9 Typo
| * 646c190 More docs related to branches
|/
* 802dca2 Fixed test for A
```

Note: This is similar to the history above.
"feature-u" shows up near the top since it
is the freshest branch. All commits show up as
before but there is an additional commit
"Copied MyServiceTest.md -> MyServiceFeatureUTest.md"
and all commit hashes have changed.

### Rewrite All Commits Of "feature-u"

We would like to rewrite most/all commits
of the branch "feature-u" with these goals:

- MyServiceTest.md ... shall NOT be modified at all
  (thus there will be no merge conflicts)
- MyServiceFeatureUTest.md ... shall contain all the
  modifications from MyServiceTest.md

We achieve this by executing:

```sh
$ git checkout feature-u
  # Save the original version of MyServiceTest.md
$ git show "$(git merge-base --all main feature-u)":MyServiceTest.md >/tmp/MyServiceTest.md
$ git filter-branch -f --tree-filter \
  'cp MyServiceTest.md MyServiceFeatureUTest.md; cp /tmp/MyServiceTest.md MyServiceTest.md;' \
  "$(git merge-base --all main feature-u)..HEAD"
WARNING: git-filter-branch has a glut of gotchas generating mangled history
	 rewrites.  Hit Ctrl-C before proceeding to abort, then use an
	 alternative filtering tool such as 'git filter-repo'
	 (https://github.com/newren/git-filter-repo/) instead.  See the
	 filter-branch manual page for more details; to squelch this warning,
	 set FILTER_BRANCH_SQUELCH_WARNING=1.
Proceeding with filter-branch...

Rewrite ba43a18650d485e05259cbe8624e520a9ded5d6f (10/10) (0 seconds passed, remaining 0 predicted)    
Ref 'refs/heads/feature-u' was rewritten
```

### Verify The Changes

```sh
$ git diff "$(git merge-base --all main feature-u)" HEAD -- MyServiceTest.md
  # No output

$ git diff "$(git merge-base --all main feature-u)" HEAD -- MyServiceFeatureUTest.md
diff --git a/MyServiceFeatureUTest.md b/MyServiceFeatureUTest.md
new file mode 100644
index 0000000..360001d
--- /dev/null
+++ b/MyServiceFeatureUTest.md
@@ -0,0 +1,91 @@
+MyServiceTest
+=============
+
+This is a markdown file representing
...
+Some implementation details of the unit test for feature V,
+modified later on and again

$ git log --oneline "$(git merge-base --all main feature-u)" HEAD -- MyServiceFeatureUTest.md
a46bd65 (HEAD -> feature-u) Work on U
538bc54 Tests for U
897e668 Tests for U
a8d54b0 Tests for U
8e3f61c Fix for V - tests
2c28bfa Started work on U
00e7f6d Copied MyServiceTest.md -> MyServiceFeatureUTest.md
```

### Cleanup "MyServiceFeatureUTest.md"

It is basically a copy of "MyServiceTest.md"
extended by the tests related to feature-u.
Delete all tests which are not related to feature-u,
since they are still within "MyServiceTest.md".

### Rebase Onto Main

```
git checkout feature-u
git rebase main
# Resolve conflicts
```

### Do A Fast-Forward For Main

```
git checkout main
git rebase main-2
git checkout feature-u
```

### Rebase Onto Main Again

```
git checkout feature-u
git rebase main
# Resolve conflicts
```

### Merge "feature-t"

```
git checkout main
git merge feature-t
# Resolve conflicts
```

### Rebase Onto Main Again

```
git checkout feature-u
git rebase main
# Resolve conflicts
```

### Merge "feature-u"

```
git checkout main
git merge feature-u
# No conflicts
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
