# git

`git` is a popular version control system for code.

*    [git-scm.com - Reference](http://git-scm.com/docs)

## Cleanup Merged Branches

```bash
git branch --merged | grep -P -v "(^\*|master|main)" | xargs git branch -d
```

## Cleanup Merged Branches - Github Workflow Fix

The above will not work for multi-commit branches that are merged via
Github "squash + merge" on a PR.

The typical workflow is:

1. Commit changes on local `dev` branch
1. Push changes to remote Github `dev` branch (same name)
1. Create Github PR to merge `dev` branch --> `main` branch
1. Get LGTM
1. Use Github squash + merge button which also deletes Github `dev` branch
1. Pull changes into local `main`
1. Running cleanup merged branches above leaves `dev` branch :(

The local `dev` branch doesn't look like it's merged because it's commit hash
is not present in the `main` log due to the squash.

### Solution 1:

```bash
git fetch -p
branches=$(git for-each-ref --format '%(refname) %(upstream:track)' refs/heads | awk '$2 == "[gone]" {sub("refs/heads/", "", $1); print $1}')
for branch in $branches;
do
  git branch -D $branch
done
git prune origin
```

Source: [Remove tracking branches no longer on remote](https://stackoverflow.com/questions/7726949/remove-tracking-branches-no-longer-on-remote/33548037#33548037)

### Solution 2:

```bash
git remote prune origin \
    | git branch  -a \
    | grep -v HEAD \
    | sed 's/remotes\/origin\///g' \
    | sed 's/^\*/ /g' \
    | sort \
    | uniq --count \
    | grep -P "^\s+1\s+" \
    | awk '{print $2}' \
    | xargs -p -L 1 git branch -D
```

Explanation: Calls `git branch -D` on any local branch not present on origin.

1. `git remote prune origin`: deletes local trackers for branches deleted on origin
1. `git branch -a`: show all tracked branches
1. `grep -v HEAD`: ignore the HEAD tracker
1. `sed 's/remotes\/origin\///g'`: remove the `remotes/origin` prefix on the remote trackers
1. `sed 's/^\*/ /g'`: remove the asterisk used to show current branch
1. `sort`: sort the lines
1. `uniq --count`: count the number of times each branch name shows up
1. `grep -P "^\s+1\s+"`: filter down to the branches which only occur once (local-only!)
1. `awk '{print $2}'`: filter out the counts
1. `xargs -p -L 1 git branch -D`:
    * `-L 1`: for each line of output (branch name)
    * `-P`: print delete branch command and request y/n prompt
    * `git branch -D`: delete the branch

### Solution 3:

This involves less complexity

```bash
# Remove remote/* branches tracked locally that have been deleted on origin.
git remote prune origin

# Show all branches tracked locally
git branch -a

# Force delete any branches which do not have a remote/* version
git branch -D $FOO
```
