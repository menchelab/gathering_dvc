# DVC Tutorial: Part 2 - Pipelines

Moving from simply tracking large files to building **DVC Pipelines** is where
the real magic happens.

Pipelines solve a very specific, very painful problem in machine learning and
data science: *"Wait, which script did I run to get this model, and what data
did I use?"* By defining pipelines, you turn your project into a Directed
Acyclic Graph (DAG) where DVC perfectly understands how your data, code, and
outputs depend on each other.

## The Goal

In Part 1, we tracked a static dataset (`data/data.csv`). Now, we are going to
write a Python script that trains a machine learning model on that data. We
will use DVC to link the data, the script, and the resulting model together
into a reproducible pipeline.

## 1. Creating the Training Script

First, let's create a simple Python script to train a Random Forest classifier
on our dataset.

Create a file named `train.py` in the root of your repository and paste the
following code into it:

```python
# train.py
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
import pickle

print("Loading data...")
df = pd.read_csv('data/data.csv')
X = df[['feature_1', 'feature_2']]
y = df['label']

print("Training model...")
clf = RandomForestClassifier(n_estimators=10, random_state=42)
clf.fit(X, y)

print("Saving model...")
with open('model.pkl', 'wb') as f:
    pickle.dump(clf, f)
    
print("Success! Model saved to model.pkl")

```

Let's commit this script to Git (since it is just lightweight code, Git handles
it perfectly):

```sh
git add train.py
git commit -m "Add training script"

```

## 2. Defining a Pipeline Stage

Instead of just running `python train.py` manually, we are going to tell DVC to
run it. We do this by creating a **pipeline stage**.

When creating a stage, we have to tell DVC three things:

1. **Dependencies (`-d`):** What files does this script need to run? (Our script and our data).
2. **Outputs (`-o`):** What heavy files does this script generate? (Our model).
3. **Command:** How do we actually execute the stage?

Run this command from the root of your repository:

```sh
dvc stage add -n train -d data/data.csv -d train.py -o model.pkl python train.py

```

*Note: `-n train` simply gives this stage the name "train".*

## 3. Understanding `dvc.yaml` and `dvc.lock`

The `dvc stage add` command didn't actually run your code yet. Instead, it
created a blueprint. Look at your repository files now—you will see a new file
called `dvc.yaml`.

Let's inspect it:

```sh
cat dvc.yaml

```

You will see human-readable instructions detailing exactly how the `train`
stage works. This file is meant to be version-controlled with Git.

Also your `.gitignore` will now contain a new entry telling git
to ignore the model file:

```sh
cat .gitignore
```

Now, let's actually execute our pipeline:

```sh
dvc repro

```

This command (`repro` stands for reproduce) tells DVC to look at `dvc.yaml` and
run any stages that need to be run. You should see the output of your Python
script in the terminal.

Because you defined an output (`-o model.pkl`), DVC automatically caches
`model.pkl` and tracks it, just like it did with `data.csv` in Part 1! It also
generates a `dvc.lock` file. This file records the exact MD5 hashes of your
dependencies and outputs at the time of execution.

Let's commit our pipeline definition to Git:

```sh
git add dvc.yaml dvc.lock .gitignore
git commit -m "Add and run training pipeline"

```

## 4. Visualizing the Pipeline

As your project grows, you might add data cleaning stages, evaluation stages,
and more. DVC can visualize how everything connects.

Run the following command:

```sh
dvc dag

```

You should see an ASCII graph showing that your `data/data.csv.dvc` file flows
directly into your `train` stage.

## 5. The Magic of Caching and Reproducibility

Here is why DVC pipelines are so powerful. Try running the pipeline again:

```sh
dvc repro

```

DVC will say: **"Stage 'train' didn't change, skipping"**.

DVC looks at the hashes in `dvc.lock`. It sees that `data.csv` hasn't changed,
and `train.py` hasn't changed. Therefore, it knows `model.pkl` is already up to
date. It skips the computation entirely, saving you immense amounts of time on
large workflows.

Let's force a change to see what happens. Open `train.py` in your text editor
and change the number of estimators in the Random Forest from `10` to `100`:

```python
# Change this line in train.py:
clf = RandomForestClassifier(n_estimators=100, random_state=42)

```

Now, ask DVC to reproduce the pipeline again:

```sh
dvc repro

```

This time, DVC detects that the hash of `train.py` has changed. It
intelligently knows that the `train` stage is now out-of-date, so it executes
the script, generates a new `model.pkl`, and updates the hashes in `dvc.lock`.

Commit the updated experiment to Git:

```sh
git add train.py dvc.lock
git commit -m "Increase n_estimators to 100"

```

## 6. Pushing the Pipeline Assets

Just like in Part 1, you want to back up your heavy files (in this case, your
newly trained `model.pkl`). Since we already configured our remote storage,
this is a one-liner:

```sh
dvc push

```

Now, anyone who clones your Git repository can simply type `dvc pull` to get
the exact `data.csv` and `model.pkl` required for your code, and type `dvc
repro` to run the entire project from scratch.
