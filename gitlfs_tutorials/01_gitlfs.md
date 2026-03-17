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

## Pushing and Pulling

Push both Git commits and LFS objects to the remote:

```sh
git push origin main
```

Git LFS objects are uploaded automatically alongside the regular push.

When cloning a repository that uses Git LFS, the large files are downloaded
automatically:

```sh
git clone <repo-url>
```

To manually fetch LFS objects for the current commit (e.g. after a bare clone):

```sh
git lfs pull
```

---

## Key Points

| Concept | detail |
|---------|--------|
| Pointer file | Small text file committed to Git in place of the large file |
| LFS server | Stores the actual file content (GitHub, GitLab, self-hosted, etc.) |
| `.gitattributes` | Records which patterns are tracked by LFS — must be committed |
| `git lfs install` | One-time setup per machine to activate LFS hooks |
