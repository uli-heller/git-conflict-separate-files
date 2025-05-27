git-conflict-separate-files
===========================

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
