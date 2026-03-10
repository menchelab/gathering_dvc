# DVC Tutorial

## Basics

The first part of this gathering will introduce you to the basic functionality
of DVC (Data Version Control).

Git is incredible for tracking code, but it notoriously chokes on large files,
binaries, or datasets. DVC solves this by working in synergy with Git. It acts
as a smart pointer system: **Git tracks the lightweight metadata files, while
DVC tracks and stores the heavy data files.**

## Installation DVC

Depending on your system and your preferences, use one of the following
commands to install DVC:

```sh
# Using uv (fast, modern python package manager)
uv tool install dvc

# Using Homebrew (MacOS/Linux)
brew install dvc

# Using pipx
pipx install dvc

# Using pip
pip install dvc

# Using Conda
conda install -c conda-forge dvc

```

## Python Environment Setup

Before generating our data, we need an environment with the right packages.
Choose the method that matches your setup:

**If you use `uv`:**

```sh
uv init 
uv add pandas matplotlib scikit-learn
source .venv/bin/activate

```

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

## Repository Setup

First, create a new folder for your repository in your preferred location and
navigate into it:

```sh
mkdir gathering_dvc
cd gathering_dvc

```

Initialize a Git repository by running:

```sh
git init 

```

Initialize DVC:

```sh
dvc init 

# If environment is not activated you can do:
uv run dvc init

```

This command creates a few hidden files that configure your DVC project. Have a
look at them:

```sh
ls -l .dvc 

```

DVC automatically stages these newly added configuration files to your next Git
commit. Let's check the status and commit them:

```sh
git status
git commit -m "Initialize DVC"

```


## Dataset Creation

Now we are ready to create an example dataset to work with. Run the following
code to create, visualize, and save a "half-moon" dataset.

*(Note: A plot window will pop up. **You must close the plot window** for the
script to finish executing and return you to the command prompt!)*

```sh
python -c "import pandas as pd; import matplotlib.pyplot as plt; from sklearn.datasets import make_moons; from pathlib import Path; Path('./data/').mkdir(parents=False, exist_ok=True); X, y = make_moons(n_samples=1000, shuffle=True, noise=0.05, random_state=42); pd.DataFrame({'feature_1': X[:, 0], 'feature_2': X[:, 1], 'label': y}).to_csv('data/data.csv', index=False); plt.scatter(X[:, 0], X[:, 1], c=y, cmap='viridis', edgecolor='k'); plt.title('Make Moons Dataset'); plt.xlabel('Feature 1'); plt.ylabel('Feature 2'); plt.show()"

```
Note: for all commands, if the env is not active and you are using uv, you need to use `uv run`

Now we created the file data.csv inside the `data` folder.

## Dataset Versioning

Now we can add the dataset for versioning via DVC, very similarly to how we add
code via Git:

```sh
dvc add data/data.csv

```

This command does the heavy lifting. It calculates the hash of `data.csv`,
moves the actual file into DVC's hidden cache, and creates two critical text
files:

1. A `.gitignore` file (telling Git to ignore the actual `.csv`).
2. A `.dvc` meta file (the pointer Git *will* track).

Let's add these pointer files to Git:

```sh
git add data/.gitignore data/data.csv.dvc
git commit -m "Add dataset V1 (half-moons)"

```

Let's peek inside the new `.gitignore` file:

```sh
cat data/.gitignore

```

As you can see, the heavy dataset will never be accidentally committed to Git
now.

Now, let's check the meta file DVC just created:

```sh
cat data/data.csv.dvc

```

Here, DVC keeps track of the MD5 hash of the file, its size, and its path. This
tiny text file is what Git uses to know exactly which version of the data
belongs to this specific commit.

## Remote Data Repository

So far, our versioning system only knows about the dataset's metadata. The
actual data is just sitting in a hidden folder (`.dvc/cache`) on your local
machine. We need a safe place to store it.

For this, we create a DVC remote repository (the exact same concept as a Git
remote, like GitHub, but for data). This remote can reside on an AWS S3 bucket,
a network drive, Google Cloud Storage, etc.

For the sake of simplicity, we will simulate this by creating a "local" remote
elsewhere on your machine. The following example creates it on a MacOS Desktop:

```sh
mkdir ~/Desktop/dvc_remote

```

Now, set this folder up as the default storage remote for DVC. *(Make sure to
replace the path if you are on Windows)*:

```sh
dvc remote add -d storage ~/Desktop/dvc_remote/

```

This command modifies your DVC config file. Let's have a look:

```sh
git diff .dvc/config

```

Commit this configuration change to Git:

```sh
git add .dvc/config
git commit -m "Configure local remote repository for DVC storage"

```

## Pushing to the Remote Repository

Now that we have configured the remote, we can push our heavy data to it:

```sh
dvc push

```

Let's verify that the data made it to the remote folder. *(If you don't have
the `tree` command installed, you can just use `ls -R ~/Desktop/dvc_remote`)*:

```sh
tree ~/Desktop/dvc_remote

```

You should see a directory structure containing a file with the exact same hash
found in your `data.csv.dvc` file. Your data is now safely backed up!

## Pulling from the Remote

Let's simulate a disaster (or simply a colleague cloning your project to a new
computer). We will delete the data from our repository, and wipe the DVC cache
so it's completely gone from our project:

```sh
rm data/data.csv
rm -rf .dvc/cache

```

If you look in your `data` folder now, `data.csv` is missing. But because Git
still has the `.dvc` pointer file, we can easily download the exact right data
back from our remote:

```sh
dvc pull

```

Check your `data` folder—`data.csv` has been perfectly restored!

## Changing the Dataset

Let's call the dataset we just restored **V1**. We will now generate a brand
new dataset (**V2**), add it to tracking, and see how seamlessly DVC lets us
time-travel between the two.

First, navigate into the data folder, run the new generation script (which uses
`make_circles` instead of `make_moons`), and then navigate back out to the
root. *(Remember to close the plot window!)*:

```sh

python -c "import pandas as pd; import matplotlib.pyplot as plt; from sklearn.datasets import make_circles; X, y = make_circles(n_samples=1000, shuffle=True, noise=0.05, factor=0.5, random_state=42); pd.DataFrame({'feature_1': X[:, 0], 'feature_2': X[:, 1], 'label': y}).to_csv('data/data.csv', index=False); plt.scatter(X[:, 0], X[:, 1], c=y, cmap='viridis', edgecolor='k'); plt.title('Make Circles Dataset'); plt.xlabel('Feature 1'); plt.ylabel('Feature 2'); plt.axis('equal'); plt.show()"

```

Because `data.csv` changed, we need to tell DVC to track the new version. From
the root of your repository:

```sh
dvc add data/data.csv

```

This updates `data/data.csv.dvc` with a brand new hash. Now, tell Git to track
that new pointer:

```sh
git add data/data.csv.dvc
git commit -m "Generate new data.csv (V2 - Circles)"

```

Push the new dataset to your remote storage:

```sh
dvc push

```

If you check your remote folder now (`tree ~/Desktop/dvc_remote`), it will
contain two different data blobs—one for V1, and one for V2.

## Switching Between Versions

To prove that we can jump back and forth, here is a quick Python script to
visually inspect whatever `data.csv` is currently in your workspace.

```sh

python -c "import pandas as pd; import matplotlib.pyplot as plt; df = pd.read_csv('data/data.csv'); plt.scatter(df['feature_1'], df['feature_2'], c=df['label'], cmap='viridis', edgecolor='k'); plt.title('Dataset Visualization from CSV'); plt.xlabel('Feature 1'); plt.ylabel('Feature 2'); plt.axis('equal'); plt.show()"

```

*Right now, this will show the **Circles (V2)** dataset.*

Let's say we realize V2 was a mistake, and we need our V1 data back. All we
have to do is tell Git to rewind our `.dvc` pointer file to the previous commit
(`HEAD~1`):

```sh
git checkout HEAD~1 data/data.csv.dvc

```

*Note: This tells Git: "Go back one commit, grab `data/data.csv.dvc` exactly as
it was back then, and put it in my current directory."*

Now that our pointer is looking at V1, we tell DVC to fetch the matching data:

```sh
dvc pull --force

```

Run the visualization script again:

```sh
python -c "import pandas as pd; import matplotlib.pyplot as plt; df = pd.read_csv('data/data.csv'); plt.scatter(df['feature_1'], df['feature_2'], c=df['label'], cmap='viridis', edgecolor='k'); plt.title('Dataset Visualization from CSV'); plt.xlabel('Feature 1'); plt.ylabel('Feature 2'); plt.axis('equal'); plt.show()"

```

*You are now looking at the **Half-Moons (V1)** dataset!*

To jump back to the future (V2), just check out the most recent pointer and
pull:

```sh
git checkout HEAD data/data.csv.dvc
dvc pull

```

## Some Discussion

As we have seen, DVC is not a standalone version control system. Instead, it
perfectly extends Git (the industry standard for code) to handle large files.

Also note that "Data" in DVC does not exclusively stand for CSVs or images. You
can use this exact same workflow to track massive deep learning model weights
(`.pt`, `.h5`), large binaries, executables, or compressed archives. If it's
too big for Git, it belongs in DVC.
