# Git Cheatsheet — Personal Notes

## Create a local repo and push to GitHub

```bash
cd /path/to/project
git init
git add .
git commit -m "Initial commit"
```

Then create an **empty** repo on GitHub (no README/license/gitignore), and connect it:

```bash
git remote add origin https://github.com/USERNAME/REPO.git
git branch -M main
git push -u origin main
```

Future pushes: just `git push`.

### What the commands mean
- `origin` — nickname/alias for the remote repo (on GitHub). Just convention.
- `main` — the default branch name. (`master` was the old default name; same concept.)
- `-M` in `git branch -M main` — force-renames the current branch to `main`. (`-M` = move/rename, uppercase forces it.)
- `-b` (different command) — `git switch -b name` *creates* a new branch. `-b` = branch.
- Remember: **M**ove = rename, **b**ranch = create.

---

## Ignore files/folders (e.g. datasets)

Create a `.gitignore` in the repo root:

```
datasets/          # ignore a folder
dataset.csv        # ignore a file
*.csv              # ignore by extension
```

```bash
echo "datasets/" >> .gitignore
git add .gitignore
git commit -m "Add gitignore for datasets"
```

### If you ALREADY committed the files
`.gitignore` doesn't untrack already-committed files. Untrack them:

```bash
git rm -r --cached datasets/    # folder (keeps files on disk)
git rm --cached dataset.csv     # single file
git commit -m "Stop tracking datasets"
git push
```

- `--cached` = remove from Git tracking but KEEP the file on disk.
- `fatal: pathspec '...' did not match any files` = it's already untracked (nothing to do).

---

## Check what Git is tracking / what's large

```bash
git ls-files | grep datasets        # is it tracked? (empty = not tracked)
git log --oneline -- datasets/       # commits that touched it (empty = not in history)
du -sh .git                          # size of Git history
du -sh .                             # total size including working files
```

Find the largest objects ever committed (the usual cause of a bloated repo):

```bash
git rev-list --objects --all |
  git cat-file --batch-check='%(objectsize:disk) %(rest)' |
  sort -rn | head -20 | numfmt --to=iec --field=1
```

**Key idea:** untracking stops tracking *going forward*, but the files stay in
*past commits* (history). That's why a push can still be huge after untracking.

---

## Remove a folder from ALL history (scrub it out)

Use when a large/unwanted folder is stuck in past commits.

```bash
# 1. (Optional but safer) back up the history
cp -r .git ../git-backup

# 2. Rewrite history to remove the folder
pip install git-filter-repo
git filter-repo --path datasets/ --invert-paths
#   --invert-paths = remove these paths, keep everything else
#   add --force if it complains "does not look like a fresh clone"

# 3. filter-repo drops the remote as a safety feature — re-add it
git remote add origin https://github.com/USERNAME/REPO.git

# 4. Make sure it's ignored so it doesn't come back
echo "datasets/" >> .gitignore
git add .gitignore
git commit -m "Add gitignore for datasets"

# 5. Force push the rewritten history
git push --force origin main
```

### Verify it worked
```bash
git log --oneline -- datasets/   # should return NOTHING
du -sh .git                      # should be much smaller
```

### Important caveats
- Your files on disk are **safe** — filter-repo only changes Git history/tracking.
- Rewriting history changes all commit hashes → requires a **force push**.
- Anyone else who cloned the repo must **re-clone** afterward.
- If `.git` is still big after rewrite, force garbage collection:
  ```bash
  git reflog expire --expire=now --all
  git gc --prune=now --aggressive
  ```
- If the data was **sensitive**, a force push isn't enough — old commit hashes
  may stay accessible on GitHub. Delete & recreate the repo, or contact GitHub
  support to purge it.

---

## Authentication note
For HTTPS pushes, GitHub asks for credentials. Use a **Personal Access Token**
(Settings → Developer settings → Personal access tokens) as the password — not
your account password. Or set up SSH and use `git@github.com:USERNAME/REPO.git`.
