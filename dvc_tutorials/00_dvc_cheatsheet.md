# DVC + Git Workflow Cheat Sheet

## 1. Setup & Initialization

Always initialize Git before DVC, as DVC needs Git to track its internal configuration files.

* `git init` — Initializes the Git repository for your code.
* `dvc init` — Initializes the DVC project, creating a `.dvc/` directory for config and cache.
* `git status` — View the newly created DVC config files.
* `git commit -m "Initialize DVC"` — Saves the initial DVC setup to Git history.

---

## 2. Tracking Large Files & Datasets

Git tracks the lightweight `.dvc` pointer files; DVC tracks the heavy data.

* `dvc add data/data.csv` — Tells DVC to track the file. This calculates the hash, moves the file to the DVC cache, and creates `data.csv.dvc` and `.gitignore`.
* `git add data/data.csv.dvc data/.gitignore` — Stages DVC's newly created pointer and ignore files so Git can track them.
* `git commit -m "Add dataset V1"` — Commits the data pointer to your project history.

---

## 3. Remote Storage & Collaboration

Set up a remote to back up your data or share it with teammates.

* `dvc remote add -d storage <path_or_url>` — Configures a default remote (e.g., local path, AWS S3, Google Cloud).
* `git add .dvc/config` — Stages the remote configuration change.
* `git commit -m "Configure DVC remote"` — Commits the remote configuration to Git.
* `dvc push` — Uploads your locally tracked large files/models to the configured remote.
* `dvc pull` — Downloads the data corresponding to your current `.dvc` pointer files from the remote.

---

## 4. Time Traveling (Switching Versions)

To change data versions, you check out the `.dvc` file in Git, then tell DVC to sync the heavy files to match.

* `git checkout HEAD~1 data/data.csv.dvc` — Reverts the data pointer file to how it was in the previous commit.
* `dvc pull` — Looks at the newly checked-out pointer and replaces your workspace data with the correct historical version.
* `git checkout HEAD data/data.csv.dvc` — Brings the pointer back to the present (most recent commit).

---

## 5. Reproducible Pipelines

Turn your scripts into a traceable Directed Acyclic Graph (DAG) using `dvc.yaml` and `dvc.lock`.

* `dvc stage add -n train -d data/data.csv -d train.py -o model.pkl python train.py` — Creates a pipeline stage named "train". It defines dependencies (`-d`), outputs (`-o`), and the execution command.
* `dvc repro` — Executes the pipeline. DVC intelligently skips stages where dependencies haven't changed. Generates/updates `dvc.lock`.
* `dvc dag` — Displays an ASCII graph in your terminal showing how your pipeline stages connect.
* `git add dvc.yaml dvc.lock .gitignore` — Stages the pipeline blueprint and the exact hashes of the current run.
* `git commit -m "Run training pipeline"` — Saves this exact reproducible experiment to Git.

---

## 6. Tracking Metrics & Parameters

Compare the performance of different model runs without leaving the terminal.

* **Define in `dvc.yaml**`: Manually add `params:` (linking to a `params.yaml` file) and `metrics:` (linking to a JSON/YAML output, setting `cache: false`).
* `dvc repro` — Reruns the pipeline using the new parameters to generate new metrics.
* `git add params.yaml dvc.lock metrics.json` — Stages your new experiment parameters, lockfile, and lightweight metrics.
* `git commit -m "Experiment: Tune hyperparameters"` — Commits the experiment to Git history.
* `dvc params diff` — Shows a clean table of which hyperparameters changed compared to your last Git commit.
* `dvc metrics diff` — Shows a comparison table of your model's performance (e.g., accuracy) between commits.

