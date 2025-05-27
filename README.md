git-conflict-separate-files
===========================

Setting Up This Repo
--------------------

```
WORK=(path-to-location-where-you-store-your-work)
OLD_PWD="$(pwd)"
cd $WORK
git init git-conflict-separate-files

cd git-conflict-separate-file
git commit --allow-empty -m "Initial commit"

# Create this file (README.md)
git add README.md

cd "${OLD_PWD}"
```
