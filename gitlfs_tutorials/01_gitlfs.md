# Git LFS Tutorial: Tracking Large Files

## Basics

Git is designed for text-based source code, and it struggles with large binary
files—every change is stored in full, bloating the repository history quickly.
**Git LFS (Large File Storage)** solves this by replacing large files in the
repository with lightweight text pointers, while the actual file contents are
stored on a separate LFS server.

When you `git clone` or `git pull`, Git LFS transparently downloads only the
large file versions you actually need, keeping your local checkout lean.

---

## Installation

Depending on your system, use one of the following commands to install Git LFS:

```sh
# Using Homebrew (macOS/Linux)
brew install git-lfs

# Using apt (Debian/Ubuntu)
sudo apt install git-lfs

# Using dnf (Fedora)
sudo dnf install git-lfs

# Using Conda
conda install -c conda-forge git-lfs
```

After installing, run the following once to set up the Git hooks that Git LFS
needs:

```sh
git lfs install
```

You only need to run `git lfs install` once per user account on a machine.

---

## Repository Setup

Initialize a Git repository (or navigate into an existing one):

```sh
mkdir gathering_dvc
cd gathering_dvc
git init
```

---

## Tracking Large Files

First let's create the data (first create your environment).

```sh
python -c "from sklearn.datasets import fetch_california_housing; import numpy as np; from pathlib import Path; housing = fetch_california_housing(); Path('./data/').mkdir(parents=False, exist_ok=True); np.savetxt('data/housing_data.txt', housing['data'])"
```

Tell Git LFS which file types (or specific files) to track. 

```sh
git lfs track "data/housing_data.txt"
```

Or we could directly to track all CSV files and model weights:

```sh
git lfs track "*.csv"
git lfs track "*.pt"
```

This creates (or updates) a `.gitattributes` file that records the tracking
rules. **This file must be committed to Git:**

```sh
git add .gitattributes
git commit -m "Configure Git LFS tracking"
```

Now add your large file as you normally would with Git:

```sh
git add data/housing_data.txt
git commit -m "Add dataset via Git LFS"
```

Git LFS automatically intercepts the add, stores the file content in the LFS
cache, and commits only the pointer to Git.

In practice, this means two different things are versioned:

- GitHub (Git remote): commits, source code, `.gitattributes`, and tiny LFS
	pointer files.
- LFS remote: the real large file bytes (datasets, model weights, binaries).

This split is the key idea of Git LFS: your Git history stays lightweight,
while large assets still remain reproducible and versioned.

---

## Verifying Tracking

Check which files are currently managed by Git LFS:

```sh
git lfs ls-files
```

To confirm that a file is stored as a pointer (and not the raw content) in Git,
inspect it:

```sh
cat data/housing_data.txt   # shows the actual content locally (LFS downloads it)
git show HEAD:data/housing_data.txt   # shows the LFS pointer stored in Git
```

---

## Local endpoint: remove and recover data

By default, Git LFS uses your Git remote as the LFS server. You can inspect the
current setup with:

```sh
git lfs env
```

For a simple test, point LFS to a folder on your machine:

```sh
mkdir -p /tmp/lfs-remote
git init --bare /tmp/lfs-remote
git config remote.origin.lfsurl file:///tmp/lfs-remote/
git config lfs.url file:///tmp/lfs-remote/
```

```sh
git config remote.origin.lfspushurl file:///tmp/lfs-remote/
```
You can verify the change by using `git lfs env` and checking the `Endpoint` field.


Now push your LFS objects to that endpoint:

```sh
git lfs push --all origin
```

Simulate data loss on your machine by removing the file and local LFS cache:

```sh
rm data/housing_data.txt
rm -rf .git/lfs/objects
```

Recover the file from the local endpoint:

```sh
git lfs pull
```

`git lfs pull` downloads the missing LFS objects and restores files for the
current checkout.

---

## File locking

File locking is useful for files that should not be edited by multiple
people at the same time.

Why it matters in teams: for binary assets, merges are often impossible or
unsafe. Locking prevents two people from modifying the same large file in
parallel and discovering the conflict only at push time.

On platforms such as Bitbucket, a locked file can usually be unlocked only by
the person who created the lock.
If you try to lock, unlock, push, or merge a file locked by someone else, the
error message typically includes the lock owner details so you can contact
them. You can also list all locks in the repository.

Important: locking does not work with `file://` endpoints (like
`file:///tmp/lfs-remote/`). Locks require an LFS API server over HTTP(S)
(GitHub, GitLab, Bitbucket, self-hosted LFS server, etc.).

If your LFS endpoint is local `file://`, skip locking.

To use locking, point LFS back to an API-backed remote and then lock:

```sh
git config --unset remote.origin.lfsurl
git config --unset remote.origin.lfspushurl
git config --unset lfs.url
git lfs locks
```

Lock a file before editing:

```sh
git lfs lock data/housing_data.txt
```

List active locks:

```sh
git lfs locks
```

Unlock:

```sh
git lfs unlock data/housing_data.txt
```

## Key points

| Concept | detail |
|---------|--------|
| Pointer file | Small text file committed to Git in place of the large file |
| LFS server | Stores the actual file content (GitHub, GitLab, self-hosted, etc.) |
| `.gitattributes` | Records which patterns are tracked by LFS — must be committed |
| `git lfs install` | One-time setup per machine to activate LFS hooks |
| `git lfs lock` | Prevents concurrent edits on lockable files |
