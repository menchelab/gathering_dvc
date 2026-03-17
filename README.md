# Data Version Control for Data Science

## Introduction

Reproducibility is a cornerstone of good science, and data science is no exception. When analyses, models, or visualizations change depending on which version of a dataset was used, results become unreliable and difficult to share. Yet most version control workflows are designed exclusively for code, leaving data as an afterthought.

Data versioning solves this by treating datasets, model weights, and other large binary artifacts with the same discipline applied to code:

- **Reproducibility:** Know exactly which version of a dataset produced a given result.
- **Collaboration:** Share data alongside code so teammates and reviewers can reproduce your work without hunting down files.
- **Auditability:** Track when data changed, why it changed, and who changed it.
- **Experimentation:** Switch between dataset versions freely, the same way you switch between git branches.

This repository contains hands-on tutorials for understanding and applying data versioning in a data science workflow. While the tooling demonstrated here is common in machine learning projects, the principles apply broadly: to any analysis pipeline, statistical model, or data product where inputs change over time.

---

## Tools for Data Versioning

Several tools exist to handle large files and data assets alongside Git. Git ususally does not work nicely with large files, so other tools are needed.

### Git LFS

[Git Large File Storage](https://git-lfs.com/) is an official Git extension that replaces large files in a repository with lightweight text pointers, while storing the actual content on a separate LFS server (GitHub, GitLab, self-hosted, etc.).

**Strengths:** Seamless Git integration

**Limitations:** Storage and bandwidth quotas on most platforms; the LFS server must stay available; not designed for frequently-changing large datasets.

### git-annex (not explained here)

[git-annex](https://git-annex.branchable.com/) takes a more decentralized approach. Files are replaced with symbolic links, and their contents can be synced across a flexible set of backends (SSH, S3, rsync, etc.) with fine-grained control over which copies live where.

**Strengths:** Very powerful and flexible; designed for offline and distributed workflows; handles arbitrarily large files.

**Limitations:** Steep learning curve; less user friendly; tooling ecosystem is smaller.

### DVC

[DVC (Data Version Control)](https://dvc.org/) is an open-source tool built specifically to extend Git for data science workflows. It stores large files in a configurable remote (S3, GCS, Azure, SSH, local path, etc.) and commits only lightweight `.dvc` pointer files to Git.

**Strengths:**
- Works with any Git repository and any remote storage backend.
- Tracks not just data but entire ML pipelines as directed acyclic graphs (DAGs), enabling reproducible runs with `dvc repro`.
- Natively tracks experiments, hyperparameters, and metrics, enabling side-by-side comparisons across commits.
- Low overhead for teams already using Git.

**Limitations:** Adds tooling complexity; requires a configured remote storage location.


## Hosting code and data for long-term access

Versioning data during active development is only part of the problem. Once a project is complete or a paper is published, the code and data need to remain accessible—sometimes for years or decades. Relying solely on platforms like GitHub for this is risky: repositories can be deleted, renamed, made private, or simply go unmaintained. Think of all the broken Github links you have encountered :broken_heart:. To solve this we can use tools that archive your data and code forever!

Dedicated archival repositories solve this by providing **persistent identifiers** (such as DOIs) and guarantees of long-term storage, independent of any individual developer's account.

### Zenodo (not explained here)

[Zenodo](https://zenodo.org/) is a general-purpose open-access research data repository hosted by CERN. Code, datasets, and other research outputs are uploaded as versioned releases and receive a permanent DOI, making them citable and reliably accessible long after a project's active development ends.

**Strengths:** Free, persistent, citable. Good for publishing research datasets and code alongside papers. Integrates directly with GitHub to archive a repository snapshot at release time.

**Limitations:** Not designed for iterative workflows; each upload is a separate release rather than a diff; not integrated with local development tooling.


## Organization

### Python Environment Setup

**If you use `uv`:**

```sh
uv init 
uv add pandas matplotlib scikit-learn
source .venv/bin/activate

```
*Note: It is not neccesary to activate the environment but remember to use uv run.*


**If you use `conda`:**

```sh
conda create -n myenv python=3.10
conda activate myenv
conda install -c conda-forge pandas matplotlib scikit-learn

```

**If you use `venv`:**

```sh
python -m venv .venv
source .venv/bin/activate
pip install pandas matplotlib scikit-learn

```
### Repo structure

This is the structure of the project.

```
gathering_dvc/
├── README.md                        # This file
├── pyproject.toml                    # All python requirements
├── dvc_tutorials/
│   ├── 00_dvc_cheatsheet.md
│   ├── 01_dvc_basics.md             
│   ├── 02_pipelines.md              
│   └── 03_metrics_parameters.md     
└── gitlfs_tutorials/
    └── 01_gitlfs.md                 
```

---

## Git lfs tutorial

Git LFS mainly works like git. There is only one tutorial.


| # | File | Topic |
|---|------|-------|
| 1 | `01_gitlfs.md` | Install Git lfs, setup a lfs remote, recover lost data, lock files. |

## DVC Tutorials

The tutorials are designed to be followed in order. Each one builds on the previous.

| # | File | Topic |
|---|------|-------|
| 1 | `01_dvc_basics.md` | Installing DVC, initializing a project, versioning a dataset, configuring a remote, and switching between data versions. |
| 1 | `01_dvc_basics.md` | Installing DVC, initializing a project, versioning a dataset, configuring a remote, and switching between data versions. |
| 2 | `02_pipelines.md` | Defining a DVC pipeline stage, running it with `dvc repro`, understanding caching and the `dvc.lock` file, and visualizing the DAG. |
| 3 | `03_metrics_parameters.md` | Externalizing hyperparameters to `params.yaml`, saving metrics to `metrics.json`, running experiments, and comparing runs with `dvc metrics diff` and `dvc params diff`. |



## Expected Outcome

Learn about data version control, hopefully include some tool in one of your projects :)


## Sources

- [The Turing Way – Version Control for Data](https://the-turing-way.netlify.app/reproducible-research/vcs)
- [DVC Documentation](https://dvc.org/doc)
- [Git LFS Documentation](https://git-lfs.com/)
- [git-annex Documentation](https://git-annex.branchable.com/)
- [Zenodo](https://zenodo.org/)
- [git lfs tutorial](https://github.com/git-lfs/git-lfs/wiki/Tutorial)

