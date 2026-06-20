# Git rebase developer notes

### Init submodules
```bash
git submodule update --init --recursive
```

### Prepare: check that we are on previous branch and all clean
```bash
git status
git -C buildroot status
```
```text
On branch rockpi_17.3
Your branch is up to date with 'origin/rockpi_17.3'.

nothing to commit, working tree clean
```

### Enable 3way diff and VSCode editor
```bash
git config --local merge.conflictStyle zdiff3
git config --global core.editor "code --wait"
```

### Add remotes:
```bash
git remote add upstream-haos https://github.com/home-assistant/operating-system.git
git -C buildroot remote add upstream-haos-buildroot https://github.com/home-assistant/buildroot.git
git remote -v
git -C buildroot remote -v
```
```text
origin  https://github.com/honsma235/haos-rockpi.git (fetch)
origin  https://github.com/honsma235/haos-rockpi.git (push)
upstream-haos   https://github.com/home-assistant/operating-system.git (fetch)
upstream-haos   https://github.com/home-assistant/operating-system.git (push)
```

### New branch:
```bash
git checkout -b rockpi_18.0
```

### Refresh upstream:
```bash
git fetch upstream-haos
```

### Start rebase (main is the current release, alternatively use a tag or hash):
```bash
git rebase upstream-haos/main
```
```text
Auto-merging buildroot-external/scripts/hdd-image.sh
CONFLICT (content): Merge conflict in buildroot-external/scripts/hdd-image.sh
error: could not apply 6a7004b73... add configurations for rock pi 4 family
...
```

now check `buildroot-external/scripts/hdd-image.sh` and merge manually, then add (stage) or
```bash
git add buildroot-external/scripts/hdd-image.sh
```

Then continue rebase:
```bash
git rebase --continue
```

### At some point we need to merge the submodule
First find the commit hash use upstream: https://github.com/home-assistant/operating-system/tree/main
```bash
cd buildroot
git checkout master
git fetch origin
git fetch upstream-haos-buildroot 89ddb15854526009aaab927a26d0cdf25c1f70f9
git rebase FETCH_HEAD
```

merge conflicts, if necessary. When done:
```bash
cd ..
git add buildroot
git rebase --continue
```

### When done, push everything (submodules must go first!)

```bash
cd buildroot
git push origin master --force-with-lease
cd ..
git push --set-upstream origin rockpi_18.0 --force-with-lease
```

### Redo something or merge commits
```bash
git rebase -i upstream-haos/main
```

Change `pick` to `edit` (or `e`) on commits you want to change in VS Code, save and close the editor tab. Make your changes in the code, then:
```bash
git add <changed-file>
git commit --amend --no-edit
git rebase --continue
git push --set-upstream origin rockpi_18.0 --force-with-lease
```
